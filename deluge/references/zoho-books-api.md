# Zoho Books API Reference — Deluge

## Base URL, Org ID, and Connection

```
booksBaseUrl = "https://www.zohoapis.com/books/v3/";
booksOrgId = "[organization_id]";          // always required as query param
// Connection name in Zoho: "zbooks"
```

**Critical difference from CRM:** Every Books API call requires `?organization_id=[orgId]` as a query parameter. There is no default org context. Define `booksOrgId` as a constant at the top of every function and append it to every URL.

```
/* CORRECT */
booksBaseUrl = "https://www.zohoapis.com/books/v3/";
booksOrgId = "123456789";
booksInvoicesUrl = booksBaseUrl + "invoices";

invoiceResponse = invokeurl
[
    url :booksInvoicesUrl + "/" + invoiceId + "?organization_id=" + booksOrgId
    type :GET
    connection:"zbooks"
];

/* WRONG — missing org_id */
invokeurl [url :booksBaseUrl + "invoices/" + invoiceId type :GET connection:"zbooks"];
```

---

## Standard Module URLs

```
booksBaseUrl       = "https://www.zohoapis.com/books/v3/";
booksInvoicesUrl   = booksBaseUrl + "invoices";
booksEstimatesUrl  = booksBaseUrl + "estimates";
booksBillsUrl      = booksBaseUrl + "bills";
booksExpensesUrl   = booksBaseUrl + "expenses";
booksContactsUrl   = booksBaseUrl + "contacts";
booksItemsUrl      = booksBaseUrl + "items";
booksVendorsUrl    = booksBaseUrl + "contacts";    // vendors are a contact type; filter by contact_type=vendor
booksPurchaseOrdersUrl = booksBaseUrl + "purchaseorders";
booksSalesOrdersUrl    = booksBaseUrl + "salesorders";
booksCreditNotesUrl    = booksBaseUrl + "creditnotes";
booksChartOfAccountsUrl = booksBaseUrl + "chartofaccounts";
booksJournalsUrl   = booksBaseUrl + "journals";
booksTaxesUrl      = booksBaseUrl + "settings/taxes";
booksCurrenciesUrl = booksBaseUrl + "settings/currencies";
```

---

## Common Endpoints

### Get a single record
```
GET [moduleUrl]/[recordId]?organization_id=[orgId]
```

### List records
```
GET [moduleUrl]?organization_id=[orgId]&page=1&per_page=200
```

### Search / filter
```
GET [moduleUrl]?organization_id=[orgId]&[filter_param]=[value]
```
Common filter params vary by module — see module-specific notes below.

### Create a record
```
POST [moduleUrl]?organization_id=[orgId]
Body: { ...fields as a map, then .toString() }
```

### Update a record
```
PUT [moduleUrl]/[recordId]?organization_id=[orgId]
Body: { ...fields to update, then .toString() }
```

### Delete a record
```
DELETE [moduleUrl]/[recordId]?organization_id=[orgId]
```

---

## Response Structure

**Single record:**
```
{
  "code": 0,
  "message": "success",
  "[entity_name]": { ...record fields }
}
```
Note: unlike CRM, the record is **not** inside a `data` array. The key is the entity name (e.g., `"invoice"`, `"bill"`, `"contact"`).

Extract with: `response.get("invoice")`

**List:**
```
{
  "code": 0,
  "message": "success",
  "[entity_name]s": [ ...records ],
  "page_context": {
    "page": 1,
    "per_page": 200,
    "has_more_page": false
  }
}
```
Extract with: `response.get("invoices")`

**Error:**
```
{
  "code": [non-zero],
  "message": "Error description"
}
```

**Standard Books response guard:**
```
booksResponseCode = bookResponse.get("code");
if(booksResponseCode != 0)
{
    errorMap.put("message","Error al extraer el registro de Books.");
    errorMap.put("details",bookResponse);
    return errorMap;
}
recordData = bookResponse.get("[entity_name]");
```

---

## Module-Specific Notes

### Invoices
- Entity key in response: `"invoice"` / `"invoices"`
- Filter params: `customer_id`, `status` (`draft`, `sent`, `overdue`, `paid`, `void`), `invoice_number`, `date`, `date_start`, `date_end`
- Line items are in `"line_items"` (list of maps)

### Bills
- Entity key: `"bill"` / `"bills"`
- Filter params: `vendor_id`, `status` (`draft`, `open`, `overdue`, `paid`, `void`), `bill_number`
- Line items: `"line_items"`

### Contacts (Customers & Vendors)
- Entity key: `"contact"` / `"contacts"`
- Filter by type: `?contact_type=customer` or `?contact_type=vendor`
- Each contact has `"contact_persons"` (list) for individual people

### Items (Products/Services)
- Entity key: `"item"` / `"items"`
- Filter: `name`, `item_type` (`sales`, `purchases`, `sales_and_purchases`)

### Purchase Orders
- Entity key: `"purchaseorder"` / `"purchaseorders"`
- Filter: `vendor_id`, `status` (`draft`, `open`, `billed`, `cancelled`)

### Estimates
- Entity key: `"estimate"` / `"estimates"`
- Filter: `customer_id`, `status` (`draft`, `sent`, `accepted`, `declined`, `invoiced`)

---

## Pagination Pattern (Books)

```
page = 1;
hasMorePages = true;
allRecords = List();

while(hasMorePages == true)
{
    pageResponse = invokeurl
    [
        url :moduleUrl + "?organization_id=" + booksOrgId + "&page=" + page + "&per_page=200"
        type :GET
        connection:"zbooks"
    ];
    pageCode = pageResponse.get("code");
    if(pageCode != 0)
    {
        errorMap.put("message","Error al obtener página " + page + " del listado.");
        errorMap.put("details",pageResponse);
        return errorMap;
    }
    records = pageResponse.get("[entity_name]s");
    if(records.isEmpty() == true || records.isNull() == true)
    {
        hasMorePages = false;
    }
    else
    {
        allRecords.addAll(records);
        pageContext = pageResponse.get("page_context");
        hasMorePages = pageContext.get("has_more_page");
        page = page + 1;
    }
}
```

---

## CRM ↔ Books Integration Notes

When writing functions that bridge CRM and Books, define both sets of constants:

```
/* CRM */
crmBaseUrl = "https://www.zohoapis.com/crm/v8/";
/* BOOKS */
booksBaseUrl = "https://www.zohoapis.com/books/v3/";
booksOrgId = "[orgId]";
```

Common pattern: fetch a CRM record → use a field value (e.g., email, custom Books ID field) → query Books with that value.
