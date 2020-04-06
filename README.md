# Data Export Tool API
API that allows you to export your JSON or XML file into Martini supported file formats: JSON, XML, YAML, CSV, and Excel

### Prerequisites

  - Apache Maven 3+
  - [Martini Desktop](https://www.torocloud.com/martini/download)

### Building the Martini Package

```
$ mvn clean package
```
This will create a ZIP file named `data-export-tool.zip` containing all the files (services, configurations, etc.) needed under the `target` folder. This ZIP file is what we call a [Martini Package](https://docs.torocloud.com/martini/latest/developing/package/) which then you can import in Martini Desktop to get started. You can learn more how to import a Martini Package by visiting our [documentation](https://docs.torocloud.com/martini/latest/developing/package/importing/)

### Getting the Martini Package from TORO Marketplace

You can also get this package via TORO Marketplace. See our documentation on [How to import a Martini Package from Marketplace](https://docs.torocloud.com/martini/latest/developing/package/importing/#from-the-marketplace).

### Usage
This package exposes operations for a simple export tool REST API that allows you to transform your JSON or XML files into Martini supported file formats. You can find the [Gloop REST API](https://docs.torocloud.com/martini/latest/developing/gloop/api/rest/) file at `/api/Converter` under the `code` folder after importing the package to your Martini Desktop application.

### Operations

The base url is `<host>/api/data-export` where `host` is the location where the Martini instance is deployed. By default, it's `localhost:8080`.

`POST /convert`

This endpoint converts a JSON or XML payload into format that is defined in the `Accept` header sent by the client e.g. `Accept: application/json` will return a JSON representation for an XML payload, and `Accept: application/xml` will return an XML representation for a JSON payload

**Sample Request**

**curl**
```
curl -X POST \
  http://localhost:8080/api/data-export/convert \
  -H 'Accept: application/xml' \
  -H 'Content-Type: application/json' \
  -H 'Postman-Token: 3eb56e95-a9f4-b17f-cda3-bb13640c7145' \
  -d '{
    "glossary": {
        "title": "example glossary",
		"GlossDiv": {
            "title": "S",
			"GlossList": {
                "GlossEntry": {
                    "ID": "SGML",
					"SortAs": "SGML",
					"GlossTerm": "Standard Generalized Markup Language",
					"Acronym": "SGML",
					"Abbrev": "ISO 8879:1986",
					"GlossDef": {
                        "para": "A meta-markup language, used to create markup languages such as DocBook.",
						"GlossSeeAlso": ["GML", "XML"]
                    },
					"GlossSee": "markup"
                }
            }
        }
    }
}'
```

**Sample Response**

Sending the request with `Accept: application/xml` header will return an xml representation of the JSON payload sent by the client.
```
<?xml version="1.0"?>
<dataOutput>
    <glossary>
        <title>example glossary</title>
        <GlossDiv>
            <title>S</title>
            <GlossList>
                <GlossEntry>
                    <ID>SGML</ID>
                    <SortAs>SGML</SortAs>
                    <GlossTerm>Standard Generalized Markup Language</GlossTerm>
                    <Acronym>SGML</Acronym>
                    <Abbrev>ISO 8879:1986</Abbrev>
                    <GlossDef>
                        <para>A meta-markup language, used to create markup languages such as DocBook.</para>
                        <GlossSeeAlso>GML</GlossSeeAlso>
                        <GlossSeeAlso>XML</GlossSeeAlso>
                    </GlossDef>
                    <GlossSee>markup</GlossSee>
                </GlossEntry>
            </GlossList>
        </GlossDiv>
    </glossary>
</dataOutput>
```

`POST /export-json`

Returns an JSON file representation of the XML payload sent in the request

**Sample Request**

**curl**
```
curl -X POST \
  http://localhost:8080/api/data-export/export-json \
  -H 'Accept: application/octet-stream' \
  -H 'Content-Type: application/xml' \
  -d '<xml>
<note>
<to>Tove</to>
<from>Jani</from>
<heading>Reminder</heading>
<body>Don'\''t forget me this weekend!</body>
</note>
</xml>'
```

**Response**

If the request is successful, the server will return a JSON file representation of the XML payload sent. The server will return the following response headers:
```
Content-Disposition →attachment; filename=eff838ad-98d1-480d-84fd-054f6d7d880c3111199946186953084.json
Content-Length →148
Content-Type →application/octet-stream;charset=UTF-8
```

`POST /export-xml`

This endpoint behaves similarly to `/export-json` endpoint. The endpoint instead accepts a JSON payload and returns an XML file repesentation.

`POST /export-csv`

Exports a JSON or XML file into CSV. A `file-type` header with value `JSON` or `XML` is required when sending a request to this endpoint. This tells the API endpoint what type of file was sent in the request.

**Note**
When converting from JSON, your JSON data must be of type array. For XML formats, it must be generic, with no attributes. See samples below:

**JSON**

```
[ {
  "item" : "No. 9 Sprockets",
  "quantity" : 12,
  "unitPrice" : 1.23
}, {
  "item" : "Widget (10mm)",
  "quantity" : 4,
  "unitPrice" : 3.45
} ]
```

**XML**

```
<?xml version="1.0"?>
<ROWSET>
    <ROW>
        <id>1</id>
        <name>Johnson, Smith, and Jones Co.</name>
        <amount>345.33</amount>
        <Remark>Pays on time</Remark>
    </ROW>
    <ROW>
        <id>2</id>
        <name>Sam "Mad Dog" Smith</name>
        <amount>993.44</amount>
        <Remark></Remark>
    </ROW>
    <ROW>
        <id>3</id>
        <name>Barney &amp; Company</name>
        <amount>0</amount>
        <Remark>Great to work with
and always pays with cash.</Remark>
    </ROW>
    <ROW>
        <id>4</id>
        <name>Johnson's Automotive</name>
        <amount>2344</amount>
        <Remark></Remark>
    </ROW>
    <ROW>
        <id>5</id>
        <name>Johnson's Automotive 2</name>
        <amount>2344</amount>
        <Remark>SUWEEEG</Remark>
    </ROW>
</ROWSET>
```

**Sample Request**

**curl**
```
curl -X POST \
  http://localhost:8080/api/data-export/export-csv \
  -H 'Cache-Control: no-cache' \
  -H 'Content-Type: multipart/form-data' \
  -H 'file-type: JSON' \
  -F multipartFile=@/path/to/json-or-xml-file
```

**Response**

If the request is successful, the server will return a CSV file representation of the file payload sent. The server will return the following response headers:
```
Content-Disposition →attachment; filename=b4e2ac7e-5372-4485-8344-82ba386d66c1.csv
Content-Length →254
Content-Type →application/octet-stream;charset=UTF-8
```

`POST /export-excel` and `POST /export-yaml`

These endpoint are similar to `/export-csv` but instead responds with an excel file, and YAML file respectively.