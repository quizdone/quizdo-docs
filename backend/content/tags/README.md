# Tags (Content API)

> [**Tags**](../../../general/components/Tags.md)

This document is only for **Content API** specifics (handlers, proto, persistence) when they exist.

## HTTP (Content service)

- **Tag groups**
  - List: `GET /tag-group/`
  - Detail: `GET /tag-group/:idOrSlug` (numeric ID or slug)
  - Create/update: `POST /tag-group/` with `CreateOrUpdateTagGroupRequest` (`proto/models/v1/tag_group.proto`, `buf.validate` enforced)
  - Delete: `DELETE /tag-group/:id` (ID-only)
- **Tags**
  - List: `GET /tag/`
  - Detail: `GET /tag/:idOrSlug` (numeric ID or slug)
  - Create/update: `POST /tag-group/:groupId/tags` with `CreateOrUpdateTagRequest` (`proto/models/v1/tag.proto`)
  - Delete: `DELETE /tag/:id` (ID-only)
- **Route rule**
  - `groupId` in `POST /tag-group/:groupId/tags` is authoritative and overrides `tag_group_id` from the body.
- **Slug rule**
  - Slug is locale-scoped (`tenant + locale + slug`) and used for readable selectors while keeping ID compatibility.

### Authorization

See [ADR 0008](../../../ADR/0008-per-resource-permission-tables.md). Permission rows are on the **tag group**; tag routes that mutate content under a group check **group** permission.

| Route | Middleware / check |
|-------|-------------------|
| `POST /tag-group/` | **EDIT** on group (`AllowEmptyURIParam` on create) |
| `DELETE /tag-group/:id` | **MANAGE** on group |
| `POST /tag-group/:groupId/tags` | **EDIT** on group (`groupId` path param) |
| `POST /tag/:id` | **EDIT** on tag |
| `DELETE /tag/:id` | **MANAGE** on tag |
| `GET` list/detail | **VIEW** |
