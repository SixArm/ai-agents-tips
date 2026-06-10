# Data Flow

**Create Flow:**

1. HTTP POST -> REST API Handler
2. Validation (required fields, format checks)
3. Duplicate Detection (search + match against existing)
4. If duplicates found: return 409 with matches
5. Repository `create()` -> Database INSERT
6. Search Engine `index_person()` -> Tantivy Index
7. Event Publisher -> Created Event
8. Audit Logger -> audit_log INSERT
9. HTTP Response -> Client

**Merge Flow:**

1. HTTP POST /merge -> REST API Handler
2. Fetch master and duplicate from database
3. Transfer data from duplicate to master
4. Update master in database
5. Soft-delete duplicate
6. Update search index
7. Publish Merged event
8. Return merge record with transferred data

**Search Flow:**

1. HTTP GET -> REST API Handler
2. Search Engine `search()` -> Tantivy Query
3. IDs -> Repository `get_by_id()` batch
4. Optional: mask sensitive data
5. Records -> JSON Serialization
6. HTTP Response -> Client (with pagination)
