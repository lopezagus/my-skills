# Zoho CRM API Reference — Deluge

## Base URL and Connection

```
crmBaseUrl = "https://www.zohoapis.com/crm/v8/";
// Connection name in Zoho: "zcrm"
```

Always use `v8` unless the user explicitly specifies a different version.

---

## Standard Module URL Pattern

```
/* Define at the top of the function */
crmBaseUrl = "https://www.zohoapis.com/crm/v8/";
crmContactsUrl     = crmBaseUrl + "Contacts";
crmLeadsUrl        = crmBaseUrl + "Leads";
crmDealsUrl        = crmBaseUrl + "Deals";
crmAccountsUrl     = crmBaseUrl + "Accounts";
crmQuotesUrl       = crmBaseUrl + "Quotes";
crmInvoicesUrl     = crmBaseUrl + "Invoices";
crmUsersUrl        = crmBaseUrl + "users";           // lowercase — system endpoint
crmSearchUrl       = crmBaseUrl + "[Module]/search";
```

Custom modules use their **API name** (usually underscored display name), e.g.:
```
crmCondicionesSalonUrl = crmBaseUrl + "Condiciones_de_Salon";
crmReglasCondicionesUrl = crmBaseUrl + "Reglas_y_Condiciones";
```

---

## Common Endpoints

### Get a single record
```
GET [moduleUrl]/[recordId]
```

### Get a list of records
```
GET [moduleUrl]?page=1&per_page=200
```

### Search records
```
GET [moduleUrl]/search?criteria=(Field_Name:equals:[value])
GET [moduleUrl]/search?criteria=((Field_1:equals:[v1])and(Field_2:equals:[v2]))
```

Search criteria syntax:
- Single condition: `(API_Field_Name:operator:value)`
- Multiple: wrap each in `()` and join with `and` or `or`
- Operators: `equals`, `not_equal`, `starts_with`, `ends_with`, `contains`, `greater_than`, `less_than`, `between`, `not_between`, `is_empty`

### Create a record
```
POST [moduleUrl]
Body: {"data":[{...fieldMap}]}
```

### Update a record
```
PUT [moduleUrl]/[recordId]
Body: {"data":[{...fieldsToUpdate}]}
```

### Delete a record
```
DELETE [moduleUrl]/[recordId]
```

---

## Response Structure

**Single record (GET by ID):**
```
{
  "data": [
    { ...record fields }
  ]
}
```
Extract with: `response.get("data").get(0)`

**List / search:**
```
{
  "data": [
    { ...record },
    { ...record }
  ],
  "info": {
    "count": 2,
    "more_records": false
  }
}
```
Iterate with `for each record in response.get("data")`

**Error:**
```
{
  "code": "INVALID_DATA",
  "details": {},
  "message": "...",
  "status": "error"
}
```

---

## Lookup / Related Field Structure

When a record has a lookup field pointing to another module, it comes back as a nested map:

```
{
  "Organizador": {
    "id": "1234567890",
    "name": "Nombre del Organizador"
  }
}
```

Extract pattern:
```
organizadorLookup = recordData.get("Organizador");
if(organizadorLookup.isNull() != true)
{
    organizadorId = organizadorLookup.get("id");
}
```

---

## Subforms

Subform data is returned as a list of maps under the subform's API name:

```
reglasList = recordData.get("Reglas_y_Condiciones");  // list of maps
for each regla in reglasList
{
    valor = regla.get("Campo_API");
}
```

---

## Common Field API Names (CRM)

Standard fields use PascalCase or specific system names:

| Display Name | API Name |
|---|---|
| Record ID | `id` |
| Record Name | `Name` |
| Owner | `Owner` (map with `id` and `name`) |
| Created Time | `Created_Time` |
| Modified Time | `Modified_Time` |
| Account Name | `Account_Name` |
| Contact Name | `Full_Name` |

Custom fields added in CRM use their API name exactly as defined in the module settings (check with the user if unsure — never guess a custom field name).

---

## Pagination Pattern

For endpoints that may return more than 200 records:

```
page = 1;
moreRecords = true;
allRecords = List();

while(moreRecords == true)
{
    pageResponse = invokeurl
    [
        url :moduleUrl + "?page=" + page + "&per_page=200"
        type :GET
        connection:"zcrm"
    ];
    pageData = pageResponse.get("data");
    if(pageData.isEmpty() == true || pageData.isNull() == true)
    {
        moreRecords = false;
    }
    else
    {
        allRecords.addAll(pageData);
        pageInfo = pageResponse.get("info");
        moreRecords = pageInfo.get("more_records");
        page = page + 1;
    }
}
```
