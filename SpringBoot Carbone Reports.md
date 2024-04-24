CarboneJS is an open source document generator, it provides:
•	The NodeJS engine generates any documents from a template and a JSON data-set.
•	Community features: Text replacement, loops, PDF generations, data formatting, and more! It does not contain Enterprise features*.
•	All export formats: DOCX, PDF, XLSX, ODS, ODG, XML, HTML, and more!

Carbone Won’t support Java.
We have used in NodeJs Project in Local. So here iam showing example with Cloud. We can call this api from our SpringBoot Application.

Example with Cloud 
	
Requirements
•	Create a Carbone Cloud account or sign-in: https://account.carbone.io/
•	For testing purposes, use a test API key: Generate unlimited watermarked PDFs for free (Only PDF file format).
•	For production purposes, use a production API key: Generate any document formats without a watermark. The free Sandbox Plan plan is available, giving you 100 credits per month. If you need more credits, you can upgrade your subscription.

Quickstart & API Flow
To generate a document, you have to:
•	Upload a template to get a template ID. The template is stored in your personal template storage.
•	Generate a report from the template ID and a JSON dataset, then the request returns a unique usable render ID.
•	Download the report thanks to the render ID.
•	To get a new document, repeat the process from point 2.
Test the API with Carbone Cloud API quickstart

Create Template
1. I have created Template in Below formate
 

2.  open https://studio.carbone.io/#/studio upload this template You will get TemplateID for It.
 


3. Test Template by Providing below data
   ```
[
        {
            "id": 3,
            "name": "Satya",
            "salary": 25000.0,
            "city": "HYD",
            "account": {
                "id": 3,
                "email": "satya@gmail.com",
                "password": "password",
                "dob": "1990-10-27"
            },
            "documentList": [
                {
                    "id": 3,
                    "proofType": "VOTERID",
                    "docName": "VoterID",
                    "docUrl": "voterIDURL"
                }
            ],
            "createdBy": null,
            "modifiedBy": null,
            "createdDate": "2023-10-27T10:59:54.891+00:00",
            "modifiedDate": "2023-10-27T10:59:54.891+00:00"
        },
        {
            "id": 4,
            "name": "Satya 2",
            "salary": 25000.0,
            "city": "HYD",
            "account": {
                "id": 4,
                "email": "satya@gmail.com",
                "password": "password",
                "dob": "1990-10-27"
            },
            "documentList": [
                {
                    "id": 4,
                    "proofType": "VOTERID",
                    "docName": "VoterID",
                    "docUrl": "voterIDURL"
                }
            ],
            "createdBy": null,
            "modifiedBy": null,
            "createdDate": "2023-10-27T11:00:00.231+00:00",
            "modifiedDate": "2023-10-27T11:00:30.571+00:00"
        }
]
   ``` 

Schema to summarize how Carbone works:
 


## Carbbone API Calls 
•	POST https://api.carbone.io/template : to push your template
•	POST https://api.carbone.io/render/:templateId : to create document
•	GET https://api.carbone.io/render/:renderId : to download final doc
To use these We need Token details. You can found in https://account.carbone.io/ in my subscriptions

CURL can be use to generate reports. The following commands can be executed on any terminal to request the Carbone Cloud API. It is containing uppercase tags that have to be replaced, such as:
•	API_KEY: Your test or production API key, login on https://account.carbone.io to get it.
•	TEMPLATE_ID: The template ID returned by the add template request.
•	RENDER_ID: The render ID returned after rendering a report.
•	FILENAME: The filename used to create the report or template just downloaded.
•	ABSOLUTE_FILE_PATH: The absolute path of a template.
•	BASE64_STRING: Template as a pure base64 string or a base64 data-URI.
TEMPLATE_ID 
5d7077481690eXXXXXXXc0c0c63cfc04472ccc9c2afd635ae66500eb0313571ec2ff022

Bearer API_KEY
test_XXXXXXXXeyJhbGciOiJFUzUxMiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiI1MzE1MjUwOTE1NjI2ODY1ODUiLCJhdWQiOiJjYXJib25lIiwiZXhwIjoyMzM1MDc3OTI5LCJkYXRhIjp7ImlkQWNjb3VudCI6IjUzMTUyNTA5MTU2MjY4NjU4NSJ9fQ.AN7CtEIaJA5FUR2qkcsAs7YvABs1OMxSqC56OgdJWsz4TfYlpwm5Ow0bMz1g8DW6PX0QDLXd3onmJfg_PLQ2LU9YAV5y3bH-8fgGU-3n1jq6mWB8wm2Fp7H3ajiidfqXxZnZzw7FhqRAYlMboTArqx0S6x8-myUU5uceQfOg8PRaa4Bv

1. Get a template with CURL
curl --location --request GET 'https://api.carbone.io/template/5d7077481690ec0c0c63cfc04472ccc9c2afd635ae66500eb0313571ec2ff022' \
--header 'carbone-version: 4' \
--header 'Authorization: test_eyJhbGciOiJFUzUxMiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiI1MzE1MjUwOTE1NjI2ODY1ODUiLCJhdWQiOiJjYXJib25lIiwiZXhwIjoyMzM1MDc3OTI5LCJkYXRhIjp7ImlkQWNjb3VudCI6IjUzMTUyNTA5MTU2MjY4NjU4NSJ9fQ.AN7CtEIaJA5FUR2qkcsAs7YvABs1OMxSqC56OgdJWsz4TfYlpwm5Ow0bMz1g8DW6PX0QDLXd3onmJfg_PLQ2LU9YAV5y3bH-8fgGU-3n1jq6mWB8wm2Fp7H3ajiidfqXxZnZzw7FhqRAYlMboTArqx0S6x8-myUU5uceQfOg8PRaa4Bv' \
-o 'FILENAME.docx'


2. Render a report with Curl
curl --location --request POST 'https://api.carbone.io/render/5d7077481690ec0c0c63cfc04472ccc9c2afd635ae66500eb0313571ec2ff022' \
--header 'carbone-version: 4' \
--header 'Content-Type: application/json' \
--header 'Authorization: test_eyJhbGciOiJFUzUxMiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiI1MzE1MjUwOTE1NjI2ODY1ODUiLCJhdWQiOiJjYXJib25lIiwiZXhwIjoyMzM1MDc3OTI5LCJkYXRhIjp7ImlkQWNjb3VudCI6IjUzMTUyNTA5MTU2MjY4NjU4NSJ9fQ.AN7CtEIaJA5FUR2qkcsAs7YvABs1OMxSqC56OgdJWsz4TfYlpwm5Ow0bMz1g8DW6PX0QDLXd3onmJfg_PLQ2LU9YAV5y3bH-8fgGU-3n1jq6mWB8wm2Fp7H3ajiidfqXxZnZzw7FhqRAYlMboTArqx0S6x8-myUU5uceQfOg8PRaa4Bv' \
--data-raw '{
    "data": [
        {
            "id": 3,
            "name": "Satya",
            "salary": 25000.0,
            "city": "HYD",
            "account": {
                "id": 3,
                "email": "satya@gmail.com",
                "password": "password",
                "dob": "1990-10-27"
            },
            "documentList": [
                {
                    "id": 3,
                    "proofType": "VOTERID",
                    "docName": "VoterID",
                    "docUrl": "voterIDURL"
                }
            ],
            "createdBy": null,
            "modifiedBy": null,
            "createdDate": "2023-10-27T10:59:54.891+00:00",
            "modifiedDate": "2023-10-27T10:59:54.891+00:00"
        },
        {
            "id": 4,
            "name": "Satya 2",
            "salary": 25000.0,
            "city": "HYD",
            "account": {
                "id": 4,
                "email": "satya@gmail.com",
                "password": "password",
                "dob": "1990-10-27"
            },
            "documentList": [
                {
                    "id": 4,
                    "proofType": "VOTERID",
                    "docName": "VoterID",
                    "docUrl": "voterIDURL"
                }
            ],
            "createdBy": null,
            "modifiedBy": null,
            "createdDate": "2023-10-27T11:00:00.231+00:00",
            "modifiedDate": "2023-10-27T11:00:30.571+00:00"
        }
    ],
    "convertTo": "pdf"
}'

{"success":true,"data":{"renderId":"MTAuMjAuMTEuNDEgICAgRRWtcLz8wl-1sP4EhNVJ5gcmVwb3J0.pdf"}}%                                                     

3. Download generated Report
curl --location --request GET 'https://api.carbone.io/render/RENDER_ID' \
--header 'carbone-version: 4' \
-o 'FILENAME.pdf'





SpringBoot Example

@GetMapping("/carbone/employee")
public ResponseEntity<Resource> employeeCarboneReport(@RequestParam("fileType") String fileType) {
    byte[] bytes = reportsService.employeeCarboneReport(fileType);
    if (null != bytes) {
        ByteArrayResource resource = new ByteArrayResource(bytes);
        String fileName = "Employee_Carbone_Report" + "_" + LocalDate.now() + ".pdf";
        return ResponseEntity.ok()
                .header(com.google.common.net.HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + fileName + "\"")
                .contentLength(resource.contentLength())
                .contentType(MediaType.APPLICATION_OCTET_STREAM)
                .body(resource);
    } else {
        throw new BusinessException("File Download Failed");
    }
}

@Data
@Builder
public class CarbonReportDto {

        @JsonProperty("data")
        private List<EmployeeDto> employeeList;

        @JsonProperty("convertTo")
        private String convertTo;

    }

@Override
public byte[] employeeCarboneReport(String fileType) {

    try {
        List<Employee> employees = employeeRepository.findAll();
        List<EmployeeDto> employeeDtos = employeeMapper.mapEntityListToDtoListForEmployee(employees);
        CarbonReportDto dto = CarbonReportDto.builder().employeeList(employeeDtos).convertTo("pdf").build();
        String renderId = getCarboneRenderId(dto);
        return downloadCarboneReport(renderId);

    } catch (JsonProcessingException e) {
        log.error("Error while converting JSON", e);
    } catch (Exception e) {
        log.error("Error while employeeCarboneReport", e);
    }
    return null;
}


private String getCarboneRenderId(CarbonReportDto dto) throws Exception{
    String renderId = null;
    String url = "https://api.carbone.io/render/" + templateID;
    HttpHeaders headers = new HttpHeaders();
    headers.set("carbone-version", "4");
    headers.setContentType(MediaType.APPLICATION_JSON);
    headers.set("Authorization", carboneToken);

    // Convert dto to JSON
    ObjectMapper objectMapper = new ObjectMapper();
    objectMapper.registerModule(new JavaTimeModule());
    String requestBody = objectMapper.writeValueAsString(dto);

    HttpEntity<String> entity = new HttpEntity<>(requestBody, headers);
    RestTemplate restTemplate = new RestTemplate();
    ResponseEntity<String> response = restTemplate.exchange(url, HttpMethod.POST, entity, String.class);
    log.info("Response: " + response.getBody());

    if(!ObjectUtils.isEmpty(response)){
        String responseBody = response.getBody();
        JSONObject jsonResponse = new JSONObject(responseBody);
        renderId = jsonResponse.getJSONObject("data").getString("renderId");
        log.info("Render Id "+renderId);
    }
    return renderId;
}


public byte[]  downloadCarboneReport(String renderId) throws IOException {
    String url = "https://api.carbone.io/render/"+renderId;
    HttpHeaders headers = new HttpHeaders();
    headers.set("carbone-version", "4");
    HttpEntity<String> entity = new HttpEntity<>(headers);
    RestTemplate restTemplate = new RestTemplate();
    ResponseEntity<byte[]> response = restTemplate.exchange(url, HttpMethod.GET, entity, byte[].class);
    return response.getBody();
}






Ref:
https://carbone.io/api-reference.html
https://studio.carbone.io/#/studio

![image](https://github.com/smlcodes/OnePageTutorials/assets/20472904/51272756-d6fb-4486-8b2c-2349d8b3d714)
