# ERP Plugin Platform — Design Spec

**Date:** 2026-06-16  
**Status:** Approved  
**Scope:** Plugin Platform Core (Phase 1) + Roadmap toàn hệ thống

---

## 1. Tổng quan

Xây dựng một **ERP webapp mở rộng được qua plugins** phục vụ nội bộ công ty. Hệ thống bao gồm một **shell app** (lõi) và các **plugin modules** (CRM, LMS, Tài chính...) có thể bật/tắt theo runtime mà không cần restart.

**Mục tiêu cốt lõi:**
- Internal tool + Business application (không phải public marketplace)
- Admin có thể bật/tắt plugin qua UI mà không cần deploy lại
- Developer có thể viết plugin mới với contract rõ ràng
- ERP scope: LMS (ưu tiên cao nhất) → Tài chính → CRM → HR → Project...

---

## 2. Tech Stack

| Tầng | Công nghệ | Ghi chú |
|------|-----------|---------|
| Frontend | React 18 + TypeScript | Vite, React Router v6 |
| Backend | NestJS + TypeScript | Dynamic Module system |
| Database | PostgreSQL | Prisma ORM |
| Cache | Redis | Session, pub/sub |
| File Storage | Cloudflare R2 | S3-compatible API |
| Auth | JWT + Refresh Token | RBAC per plugin |

---

## 3. Plugin Architecture

### Cơ chế hoạt động: Dynamic Runtime

Plugin được bật/tắt **trong khi app đang chạy** — trạng thái kích hoạt lưu trong database.

> **Clarification quan trọng:**
> - **Frontend (React.lazy):** Load thực sự ở runtime — bật plugin → component load ngay, không reload trang. ✅
> - **Backend (NestJS):** NestJS load DynamicModules tại **startup** theo danh sách từ DB. Toggle on/off plugin đã installed chỉ ảnh hưởng frontend. Cài plugin **mới từ Git** (có server module) cần 1 lần restart backend sau khi clone xong. Đây là trade-off chấp nhận được.
> - **Kết quả thực tế:** Các plugin "local" đã bundle sẵn → bật/tắt hoàn toàn không restart. Git plugin mới → cần restart 1 lần sau install.

### Plugin Registry: DB-backed

Plugin files nằm trên filesystem (`/plugins/*`), metadata (enabled, config, version) lưu trong bảng `plugins` của PostgreSQL. App load danh sách plugin enabled từ DB khi khởi động.

### Nguồn plugin

- **Local:** Thư mục `/plugins/<name>/` trong codebase — deploy cùng app
- **Git-based:** Admin paste Git URL vào Admin UI → server clone vào `/plugins/` → đăng ký vào DB

---

## 4. Plugin Contract

### Cấu trúc thư mục

```
/plugins
  /crm
    manifest.json      ← metadata và khai báo extension points
    index.tsx          ← React entry point
    /server
      index.ts         ← NestJS DynamicModule entry
      /controllers
      /services
      /entities
```

### manifest.json

```json
{
  "id": "crm",
  "name": "CRM",
  "version": "1.0.0",
  "description": "Quản lý khách hàng và pipeline bán hàng",
  "source": "local",
  "entry": "./index.tsx",
  "server": "./server/index.ts",
  "permissions": ["users:read", "contacts:read", "contacts:write"],
  "hooks": {
    "sidebar": {
      "label": "CRM",
      "icon": "👥",
      "order": 20
    },
    "slots": ["dashboard.top", "user.profile.actions"],
    "routes": ["/crm", "/crm/:id"]
  }
}
```

### Frontend Plugin API

```typescript
// Plugin nhận context từ Shell thông qua PluginContext
interface PluginContext {
  registerPage(path: string, component: React.ComponentType): void;
  registerSlot(slotId: string, component: React.ComponentType): void;
  addSidebarItem(item: SidebarItem): void;
  onEvent(event: string, handler: EventHandler): void;
  emitEvent(event: string, payload: unknown): void;
  addContextMenu(target: string, items: ContextMenuItem[]): void;
  getCurrentUser(): User;
  hasPermission(permission: string): boolean;
}

// Plugin export một init function
export default function init(ctx: PluginContext) {
  ctx.registerPage('/crm', CRMPage);
  ctx.registerSlot('dashboard.top', RevenueWidget);
  ctx.addSidebarItem({ label: 'CRM', icon: '👥', path: '/crm', order: 20 });
  ctx.onEvent('contact.created', handleNewContact);
}
```

---

## 5. Kiến trúc Tổng thể (Layer Diagram)

```
┌─────────────────────────────────────────────────────┐
│  Browser — React App                                 │
│  Shell App → Plugin Loader → Slot Renderer → Event Bus│
└──────────────────┬──────────────────────────────────┘
                   │ REST API / WebSocket
┌──────────────────▼──────────────────────────────────┐
│  Backend — NestJS                                    │
│  Plugin Registry Module → Dynamic Module Loader      │
│  Hook Manager → Auth/RBAC → Plugin APIs              │
└──────────────────┬──────────────────────────────────┘
                   │ Plugin API Contract
┌──────────────────▼──────────────────────────────────┐
│  Plugins (Local + Git)                               │
│  [built-in: core-ui, auth]                           │
│  [local: crm, lms, project]                          │
│  [git: hr, accounting]                               │
└──────────────────┬──────────────────────────────────┘
                   │ Prisma ORM
┌──────────────────▼──────────────────────────────────┐
│  Infrastructure                                      │
│  PostgreSQL · Redis · Cloudflare R2                  │
└─────────────────────────────────────────────────────┘
```

---

## 6. Plugin Boot Flow

1. App khởi động → Backend query DB lấy danh sách `plugins WHERE enabled = true`
2. Backend load NestJS `DynamicModule` cho từng plugin enabled
3. Frontend gọi `GET /api/plugins/active` nhận danh sách manifest
4. Plugin Loader dùng `React.lazy()` + dynamic `import()` load bundle từng plugin
5. Mỗi plugin gọi `init(ctx)` → đăng ký pages, slots, sidebar items
6. Shell render sidebar và dashboard slots theo kết quả đăng ký

---

## 7. UI Extension System

### Slots (vị trí inject component)

| Slot ID | Vị trí |
|---------|--------|
| `dashboard.top` | Widgets hàng đầu dashboard |
| `dashboard.bottom` | Widgets hàng dưới dashboard |
| `sidebar.top` | Trên cùng sidebar (dưới logo) |
| `user.profile.actions` | Action buttons ở trang profile user |
| `global.header.actions` | Nút trên thanh header |

### Event Bus (plugin-to-plugin communication)

Plugin giao tiếp qua event bus — không import trực tiếp lẫn nhau:

```
crm.emitEvent('contact.created', { id, name })
lms.onEvent('contact.created', linkToStudent)
```

---

## 8. Database Schema (Plugin Registry)

```sql
-- Bảng đăng ký plugin
CREATE TABLE plugins (
  id          VARCHAR PRIMARY KEY,        -- "crm", "lms"
  name        VARCHAR NOT NULL,
  version     VARCHAR NOT NULL,
  source      VARCHAR NOT NULL,           -- "local" | "git"
  git_url     VARCHAR,                   -- null nếu local
  enabled     BOOLEAN DEFAULT false,
  config      JSONB DEFAULT '{}',        -- per-plugin settings
  installed_at TIMESTAMP DEFAULT NOW(),
  updated_at  TIMESTAMP DEFAULT NOW()
);
```

---

## 9. Admin UI — Plugin Manager

Trang `/admin/plugins` cho phép:

- **Danh sách plugin** — hiển thị tên, version, source, trạng thái
- **Toggle bật/tắt** — gọi API `PATCH /api/plugins/:id/toggle`, backend hot-reload module
- **Config per plugin** — JSON editor cho plugin settings
- **Cài từ Git** — paste URL, server clone + register vào DB
- **Logs** — xem activity log của từng plugin
- **Remove** — xóa plugin khỏi DB (file còn trên disk)

---

## 10. RBAC (Role-Based Access Control)

RBAC là cross-cutting concern — được build như một **built-in module** (không phải plugin) trong Phase 1:

- `Role`: Admin, Manager, Staff, Guest
- `Permission`: string dạng `resource:action` (e.g. `contacts:write`)
- Plugin khai báo permissions cần trong `manifest.json`
- Shell kiểm tra `hasPermission()` trước khi render plugin routes/slots

---

## 11. Roadmap

### Phase 1 — Plugin Platform Core + Auth
- [ ] Shell App (React + Vite): layout, router, sidebar động
- [ ] Plugin Loader: React.lazy + dynamic import
- [ ] Slot Renderer: slot registry, inject components
- [ ] Event Bus: pub/sub in-process
- [ ] Plugin Registry API (NestJS): CRUD, toggle, git install
- [ ] Plugin DB table + Prisma migration
- [ ] Auth module (built-in): JWT, refresh token, session
- [ ] RBAC module (built-in): roles, permissions, `hasPermission()`
- [ ] Admin UI: Plugin Manager page
- [ ] Plugin mẫu: `core-dashboard` (dashboard trống với slots)

### Phase 2 — LMS Plugin *(nhiệm vụ chính)*
- [ ] Spec riêng: `2026-XX-XX-lms-plugin-design.md`
- Quản lý khóa học, bài giảng, học viên, tiến độ học

### Phase 3 — Finance Plugin
- [ ] Spec riêng: `2026-XX-XX-finance-plugin-design.md`
- Thu chi, báo cáo tài chính, học phí (liên kết với LMS)

### Phase 4 — CRM Plugin
- [ ] Quản lý khách hàng, lead, pipeline

### Phase 5+ — HR, Project Management...

---

## 12. Verification Plan

### Phase 1 Done khi:
1. Developer có thể tạo folder `/plugins/hello-world/` với `manifest.json` + `index.tsx` → plugin xuất hiện trong Admin UI
2. Admin bật plugin → sidebar item và dashboard slot hiển thị không cần restart
3. Admin tắt plugin → mọi UI của plugin biến mất
4. Admin paste git URL → plugin được clone và cài thành công
5. RBAC: user không có permission không thể truy cập route của plugin

### Testing Strategy
- Unit test: Plugin Registry service (toggle, install, validate manifest)
- Integration test: Boot flow (DB → load modules → API response)
- E2E test (Playwright): Admin UI toggle flow
