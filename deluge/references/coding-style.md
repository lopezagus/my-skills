# Coding Style Reference — Deluge House Style

This file defines the canonical coding style. Every generated or reviewed function must conform to it.

---

## 1. Variable Naming

**Always camelCase.** No snake_case, no PascalCase (except for map/list literals when used as named constants).

```
/* CORRECT */
crmBaseUrl = "https://www.zohoapis.com/crm/v8/";
quoteRequestResponse = invokeurl [...];
organizadorResponseData = organizadorRequestResponse.get("data");

/* WRONG */
crm_base_url = ...
QuoteRequestResponse = ...
```

---

## 2. URL Construction Pattern

Define a base URL constant first. Build module URLs by concatenating the base. Build record-specific URLs by concatenating the module URL with the ID.

```
/* BASE */
crmBaseUrl = "https://www.zohoapis.com/crm/v8/";

/* MODULE LEVEL */
crmQuotesUrl = crmBaseUrl + "Quotes";
crmContactsUrl = crmBaseUrl + "Contacts";

/* RECORD LEVEL — built inline when needed */
quoteRequestResponse = invokeurl
[
    url :crmQuotesUrl + "/" + recordId
    type :GET
    connection:"zcrm"
];
```

Never hardcode a full URL more than once. If a URL is used more than once, it gets its own named variable.

---

## 3. Execution Flags

Every function must declare `logMode` and `logRaw` at the top of the constants block.

```
logMode = true;
logRaw = false;
```

**logRaw = true** → every raw API response is printed to the console via `info`.  
**logMode = true** → only summarized or meaningful intermediate values are printed.

Apply them with guards:
```
if(logRaw == true)
{
    info quoteRequestResponse;
}
if(logMode == true)
{
    info "Total de artículos: " + totalArticulos;
}
```

---

## 4. Return Maps

Initialize `errorMap` and `successMap` at the top of the constants block:

```
errorMap = {"code":500,"message":null,"details":null};
successMap = {"code":0,"message":null,"details":null};
```

**On error:** populate `message` and `details`, then `return errorMap`.  
**On success:** populate `message` and `details` if relevant, then `return successMap`.

```
errorMap.put("message","Error al extraer los datos de la Cotización.");
errorMap.put("details",quoteRequestResponse);
return errorMap;
```

---

## 5. Comment Style

**Block comments only.** No inline `//` comments.

### Section header block (for major sections):
```
/*
# --------------------------------------------------------------------------------------------------------------
#
# NOMBRE DE LA SECCIÓN
#
# Descripción de qué hace esta sección. Puede ser larga si la lógica es compleja.
#
# --------------------------------------------------------------------------------------------------------------
*/
```

### Short label block (for subsections or named groups of variables):
```
/* VARIABLES CONSTANTES DE ZOHO */
/* EXTRACCION DE LOS DATOS DE LA COTIZACIÓN */
/* MANEJO DE ERRORES */
```

Never add inline comments like `crmBaseUrl = "..." // base url for CRM`. The variable name should be self-documenting.

---

## 6. API Call Pattern

Every `invokeurl` block must be followed immediately by a null/empty guard. No exception.

```
recordResponse = invokeurl
[
    url :moduleUrl + "/" + recordId
    type :GET
    connection:"zcrm"
];
if(logRaw == true)
{
    info recordResponse;
}
recordData = recordResponse.get("data");
if(recordData.isEmpty() == true || recordData.isNull() == true)
{
    errorMap.put("message","Error al extraer los datos del registro.");
    errorMap.put("details",recordResponse);
    return errorMap;
}
else
{
    recordData = recordData.get(0);
}
```

For list endpoints (no single record), omit the `.get(0)` and iterate directly.

---

## 7. Braces and Indentation

Always use `{}` braces, even for single-line `if` bodies. Use tab indentation. Align braces under the keyword.

```
if(condition == true)
{
    doSomething();
}
```

---

## 8. Full Function Structure Template

```
/*
# --------------------------------------------------------------------------------------------------------------
#
# DOCUMENTACIÓN DE FUNCIÓN
#
# [Descripción del objetivo de la función, qué módulos toca, qué devuelve.]
#
# --------------------------------------------------------------------------------------------------------------
#
# VARIABLES CONSTANTES Y EXTRACCIÓN DE DATOS
#
# [Descripción breve de las constantes definidas abajo.]
#
# --------------------------------------------------------------------------------------------------------------
*/

/* VARIABLES CONSTANTES DE [APP] */
[baseUrl] = "[base API url]";
[moduleUrl] = [baseUrl] + "[ModuleName]";

/* VARIABLES DE EJECUCIÓN */
logMode = true;
logRaw = false;
errorMap = {"code":500,"message":null,"details":null};
successMap = {"code":0,"message":null,"details":null};

/*
# --------------------------------------------------------------------------------------------------------------
#
# EXTRACCIÓN DE DATOS
#
# [Descripción de la secuencia de extracciones.]
#
*/

/* [NOMBRE DEL PRIMER RECURSO A EXTRAER] */
[response] = invokeurl
[
    url :[moduleUrl] + "/" + id
    type :GET
    connection:"[connectionName]"
];
if(logRaw == true)
{
    info [response];
}
[responseData] = [response].get("data");
if([responseData].isEmpty() == true || [responseData].isNull() == true)
{
    errorMap.put("message","[Mensaje de error descriptivo].");
    errorMap.put("details",[response]);
    return errorMap;
}
else
{
    [responseData] = [responseData].get(0);
}

/*
# --------------------------------------------------------------------------------------------------------------
#
# PROCESAMIENTO
#
*/

[... logic here ...]

successMap.put("message","[Descripción del resultado exitoso].");
successMap.put("details",[resultadoFinal]);
return successMap;
```
