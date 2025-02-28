## PLIR dockerized setup
This project contains the dockerized setup for the [PLIR](https://wiki.openmrs.org/pages/viewpage.action?pageId=235278351) project Proof Of Concept.
see [PLIR archtecture](https://wiki.openmrs.org/display/projects/Architectural+Design+Approach+to+support+an+integrated+approach+to+patient-level+indicator+reporting+for+OpenMRS)

Ensure you have Docker and Docker compose installed locally.
See Docker Installation Instructions for your environment.  If running on Linux, check https://docs.docker.com/compose/install/ to install docker compose.

To run the project , follow the instructions below .
1. Clone the repository locally

        git clone https://github.com/openmrs/openmrs-plir-dockerized-setup.git

2. Move to the project root directory and spin up the pre-configured OpenMRS ,OpenHIM ,Hapi-Fhir and the streaming-binlog pipeline instances . 

       ./run.sh

3. You should be able to acces the OpenMRS ,OpenHIM and Hapi-Fhir instances  at the following urls



| Instance  |     URL       | credentials (user : password)|
|---------- |:-------------:|------:                       |
| OpenMRS   |  http://localhost:8080/openmrs  | admin : Admin123 |
| OpenHIM   |    http://localhost:9000  |  root@openhim.org : openhim-password |
| Hapi FHir | http://localhost:8090 |    hapi : hapi123| 


   Note:
 * The OpenMRS Instance is pre-loaded with CIEL and a  sample form (HIV_form) to collect TX_PVLS specific data

 * The above script also loads the necesary TX_PVLS Measure and Library Resources into the Hapi FHir . see the Resources under the `resources` folder.

 * The Openhim instance is pre-configured with the necesary meta-data

  > Wait for the **plir-streaming-pipeline** container to start running before adding any data into OpenMRS. The running Pipeline will then listen to any  data changes  added in to OpenMRS and route them to the FHIR server through OpenHIM.

   

  4. Invoke the **collect-data** FHIR Operation using the GET request below to generate the relevant Dataset for TX_PVLS


          GET: http://localhost:8090/fhir/Measure/TX-PVLS/$collect-data?periodStart=<date>&periodEnd=<date>
   

  5. Invoke the  **evaluate-measure** FHIR Operation using the GET request below for the  indicator calculation based on CQL evaluation

         GET: http://localhost:8090/fhir/Measure/TX-PVLS/$evaluate-measure?periodStart=<date>&periodEnd=<date> 
         
  > Note : substitute `<date>` in the GET request with your actual date parameter  .  

   6. You can use a [plir-widget](https://github.com/mozzy11/plir-widget) innitially develeoped by the bahmni-plir team , to easily query and  visualise the Measure report generated from the HAPI-FHIR server .
   
* install the [http-server](https://www.npmjs.com/package/http-server)  .
 see inginix (proxy-server) config under  `plir-widget/config/nginx.conf`  . 
* To run the widget , run the command below from the root directory



          ./run-widget.sh  


 * you should be able to access the widget at `http://localhost:7000/` .

>  Procede to select the measure  TX-PVLS ,
      select the start date and generate the report using the widget
      


 7. To remove and clean out all created containers and volumes, run

          ./stop.sh

 ## Auto Regenerating the TX_PVLS Library resource    
 In case changes have been made to the Library cql logic at  `resources/cql_logic.txt`  , the  TX_PVLS Library resource  at  `resources/tx_pvls-library.json`  can be auto re-generated .

 * install the [jq](https://linuxhint.com/bash_jq_command/) library 

 * run the command below from the root directory

       ./regenerate-library.sh
  > The above script encodes the cql logic into Base64 format and rebuilds the TX_PVLS Library     

## Main Repositories
* Hapi FHir for OpenMRS PLIR https://github.com/openmrs/openmrs-contrib-plm-fhir-server
* Analytics Engine Pipeline   https://github.com/GoogleCloudPlatform/openmrs-fhir-analytics



