
# Unified Object Envelope Specification (Strict) — v1.0

## Overview
This specification defines a strict, server‑controlled data envelope for transporting objects between services.  
Every object MUST follow the exact structure below:

```
{
  "identity": { ... },
  "data": { ... },
  "related": { ... },
  "meta": { ... },
  "actions": { ... }
}
```

All sections are mandatory.  
If a section contains no information, it MUST still appear as an empty object (`{}`).

---

# 1. Identity Block

The **identity** block MUST contain immutable identifiers and classification fields.

### Required fields:
- `id` — unique identifier  
- `type` — canonical resource type  
- `displayName` — human‑friendly name  

### Optional standardized fields:
- `status` — active / inactive / archived  
- `slug` — stable human‑friendly identifier  

### Rules:
- Contains only identifiers.  
- No domain attributes, meta, or relations.  
- `id` MUST be immutable.

### Example:
```json
"identity": {
  "id": "123",
  "type": "strain",
  "displayName": "Strain ABC",
  "status": "active"
}
```

---

# 2. Data Block

The **data** block contains pure attributes of the entity.

### Allowed:
- strings  
- numbers  
- booleans  
- dates  
- short lists of scalars  
- computed summary fields  

### Rules:
- NO IDs.  
- NO foreign keys.  
- NO relationship info.  
- NO UI-specific fields.  

### Example:
```json
"data": {
  "description": "Used for genotyping",
  "dateAdded": "2025-11-12",
  "assayCount": 14
}
```

---

# 3. Related Block

The **related** block contains ONLY relationships.

Three allowed formats:

### 1. Shallow single reference:
```json
"user": { "id": "45", "type": "user" }
```

### 2. Shallow list reference:
```json
"assays": [
  { "id": "1", "type": "assay" }
]
```

### 3. Embedded related object:
This object must follow the SAME envelope specification.

```json
"user": {
  "identity": { ... },
  "data": { ... },
  "related": {},
  "meta": {},
  "actions": {}
}
```

### Rules:
- MUST contain only references or embedded envelopes.  
- NO attributes.  
- NO primitive values.  
- Must always be present (empty object allowed).

### Example:
```json
"related": {
  "user": { "id": "45", "type": "user" },
  "assays": [
    { "id": "1", "type": "assay" }
  ]
}
```

---

# 4. Meta Block

The **meta** block contains system and transport metadata.

### Required:
- `schemaVersion`  
- `serverTime` (UTC timestamp)  
- `etag` (hash for caching/versioning)  
- `permissions` (object with Boolean flags)

### Optional:
- diagnostics (`queryTimeMs`, `dbTimeMs`)  
- caching hints  
- pagination cursors  

### Rules:
- Contains NO business data.  
- Contains NO display labels.  
- Only technical metadata.

### Example:
```json
"meta": {
  "schemaVersion": "1.0",
  "serverTime": "2025-11-25T14:30:00Z",
  "etag": "b39f...f1",
  "permissions": {
    "canUpdate": true,
    "canDelete": false
  }
}
```

---

# 5. Actions Block

The **actions** block exposes hypermedia affordances.

### Required fields per action:
- `name`  
- `method`  
- `href`  

### Optional:
- `requires` (permission dependencies)  
- `enabled` (true/false)

### Rules:
- Must exist even if empty.  
- If the user lacks permission, the action MUST still appear with `"enabled": false`.  
- No domain logic inside actions.  

### Example:
```json
"actions": {
  "update": {
    "name": "update",
    "method": "PATCH",
    "href": "/strains/123",
    "requires": ["canUpdate"],
    "enabled": true
  },
  "delete": {
    "name": "delete",
    "method": "DELETE",
    "href": "/strains/123",
    "enabled": false
  }
}
```

---

# 6. Full Example Object

```json
{
  "identity": {
    "id": "123",
    "type": "strain",
    "displayName": "Strain ABC",
    "status": "active"
  },
  "data": {
    "description": "Used for genotyping",
    "dateAdded": "2025-11-12",
    "assayCount": 14
  },
  "related": {
    "user": { "id": "45", "type": "user" },
    "assays": [
      { "id": "1", "type": "assay" }
    ]
  },
  "meta": {
    "schemaVersion": "1.0",
    "serverTime": "2025-11-25T14:30:00Z",
    "etag": "b39f...f1",
    "permissions": {
      "canUpdate": true,
      "canDelete": false
    }
  },
  "actions": {
    "update": {
      "name": "update",
      "method": "PATCH",
      "href": "/strains/123",
      "enabled": true
    },
    "delete": {
      "name": "delete",
      "method": "DELETE",
      "href": "/strains/123",
      "enabled": false
    }
  }
}
```

---

# 7. Validation Guarantees

This envelope provides:
- predictable UI rendering  
- consistent microservice communication  
- safe versioning  
- strict separation between identity/data/relations/actions  
- compatibility with JSON Schema  
- server‑controlled data shape

---

# 8. Change Log

### v1.0 (Initial Release)
- Introduced 5-block strict envelope  
- Enforced required presence of all blocks  
- Defined rules for identity, data, related, meta, and actions  
- Provided full examples  
- Prepared for JSON Schema generation in future versions

---

# End of Specification
