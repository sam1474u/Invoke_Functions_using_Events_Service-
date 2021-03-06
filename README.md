

Invoke Functions using Events Service 

## Objectives
  1. Create an application.
  2. Create a Dynamic Group.
  3. Create Object Storage Bucket.
  4. Create an Autonomous Data Warehouse Database.
  5. Deploy a Function.
  6. Create an Event rule.
  7. Test the created Function
  # Prerequisites
  1. Your Oracle Cloud Trial Account
  2. Completed the Prerequisites for Functions [Markdown - Link](#https://www.oracle.com/webfolder/technetwork/tutorials/infographics/oci_functions_cloudshell_quickview/functions_quickview_top/functions_quickview/index.html
)

## STEP 1: Create an application
  In this step, you will create an application and set up Fn CLI on Cloud Shell.
  1.	Under Solutions and Platform, select Developer Services and click Functions.
  2.	Select your development compartment from the Compartment list.
  3.	Click Create Application.
  4.	For name, enter etl-app.
  5.	Select the VNC you created earlier (e.g. fn-vcn).
  6.	Select the public subnet.
  7.	Click Create.
  8.	Click on the created application to open the application details.
   
  <p><figure> <img  src="https://user-images.githubusercontent.com/42166489/108214698-7a6caa00-7156-11eb-8f48-1cc01c940a74.png"></img></figure></p>
  
## STEP 2: Create a Dynamic Group
  To use other OCI Services, your function must be part of a dynamic group. For information on creating dynamic groups, refer to the documentation.
  Before you create a dynamic group, you need to get your development compartment OCID. You will use the compartment OCID in the dynamic group matching rule.
  <p><figure> <img   src="https://user-images.githubusercontent.com/42166489/108214804-9d975980-7156-11eb-97e2-b5adb6365219.png"></img></figure></p>
  1.	Open the navigation menu, select Identity, and then Compartments.
  2.	Find your development compartment from the list, hover over the cell in the OCID column and click Copy, to copy the compartment OCID to your clipboard.
  3.	Store the compartment OCID as you will use it soon.
  Now you're ready to create a dynamic group.
  <p><figure><img  src="https://user-images.githubusercontent.com/42166489/108214886-b4d64700-7156-11eb-8ee5-5e489c2aa1b4.png"></img></figure></p>

  4.	To create a dynamic group, open the navigation menu, select Identity, and then Dynamic Groups.
  5.	Click Create Dynamic Group.
  6.	For name, enter functions-dynamic-group.
  7.	For description, enter Group with all functions in a compartment.
  8.	To select the functions that belong to the dynamic group, write matching rules. Write the following matching rule that includes all functions within a compartment you      created your application in:
  <p><figure><img src="https://user-images.githubusercontent.com/42166489/108214923-bdc71880-7156-11eb-9243-7ffa68a45610.png"></img></figure></p>

    {resource.type = 'fnfunc', resource.compartment.id = 'ocid1.compartment.oc1..example'}
    Note: Make sure you replace the above value with the compartment OCID you stored earlier.

## STEP 3: Create Object Storage Bucket

  You need a input-bucket bucket in Object Storage. You will use the input-bucket to drop-in the CSV files. The function will process the file and import them into Autonomous   Datawarehouse.
  Let's create the input-bucket first:

  1.	Open the navigation menu, select Object Storage, and then select Object Storage.
  2.	Select your development compartment from the Compartment list.
  3.	Click the Create Bucket.
  4.	Name the bucket input-bucket.
  5.	Select the Standard storage tier.
  6.	Check the Emit Object Events check box.
  7.	Click Create Bucket.
 <p><figure><img src= "https://user-images.githubusercontent.com/42166489/108215100-f0711100-7156-11eb-926e-50c56c671373.png"></img></figure></p>

 
## STEP 4: Create IAM policies
  Create a new policy that allows the dynamic group (functions-dynamic-group) to manage objects in the bucket.

  1.	Open the navigation menu, select Identity, and then select Policies. 
  2.	Click Create Policy.
  3.	For name, enter functions-buckets-policy.
  4.	For description, enter Policy that allows functions dynamic group to manage objects in the bucket.
  5.	Click the Customize (Advanced) link and paste the policy statements into the Policy Builder field:
  
       Allow dynamic-group functions-dynamic-group to manage objects in compartment [compartment-name] where target.bucket.name='input-bucket'
       Note: replace the compartment-name with the name of your development compartment (the one where you created the VCN and Function Application).
       
  6.	Click Create.
 <p><figure><img src= "https://user-images.githubusercontent.com/42166489/108215203-11396680-7157-11eb-97a5-0373767ea43e.png"></img></figure></p>


  

## STEP 5: Create an Autonomous Data Warehouse
  The function accesses the Autonomous Database using SODA (Simple Oracle Document Access) for simplicity. You can use the other type of access by modifying the function.
 <p><figure><img src= "https://user-images.githubusercontent.com/42166489/108230427-c889a980-7166-11eb-80a7-fd0c57e4e15a.png"></img></figure></p>

  1.	Open the navigation menu, select Autonomous Data Warehouse.
  2.	Click Create Autonomous Database.
  3.	From the list, select your development compartment.
  4.	For display name and database name, enter funcdb.
  5.	For the workload type, select Transaction Processing.
  6.	For deployment type, select Shared Infrastructure.
  7.	Enter the admin password.
  8.	Select BYOL
  9.	Click Create Autonomous Database.
  <p><figure><img src= "https://user-images.githubusercontent.com/42166489/108230538-e6efa500-7166-11eb-98d1-499ed8b4de13.png"></img></figure></p>

  Wait for OCI to provision the Autonomous Database, and then click the Service Console button.
   <p><figure><img src= "https://user-images.githubusercontent.com/42166489/108231028-5d8ca280-7167-11eb-9eb5-b8895ebe3254.png"></img></figure></p>


  1.	Click Development from the sidebar. 
  2.	Under RESTful Services and SODA, click Copy URL.  <br/><p><figure><img src="https://user-images.githubusercontent.com/42166489/108231160-7c8b3480-7167-11eb-9856-66fdccc4c7d9.png"></img></figure></p>

  3.	From your terminal (or Cloud Shell), create the collection called regionsnumbers by running the command below. Make sure you replace the <ORDS_BASE_URL> with the value you copied in the previous step, and <DB-PASSWORD> with the admin password you set when you created the Autonomous Database.
  4.	export ORDS_BASE_URL=<ORDS_BASE_URL>
  5.	curl -X PUT -u 'ADMIN:<DB-PASSWORD>' -H "Content-Type: application/json" $ORDS_BASE_URL/admin/soda/latest/regionsnumbers
  6.	To double check collection was created, you can list all collections. The output should look similar as below:
  7.	$ curl -u 'ADMIN:<DB-password>' -H "Content-Type: application/json" $ORDS_BASE_URL/admin/soda/latest/
  <p><figure><img src="https://user-images.githubusercontent.com/42166489/108231382-b6f4d180-7167-11eb-82f9-2a274bd64866.png"></img></figure></p>

  
## STEP 6: Deploy the function

  In this step, you will clone the functions source code repository and use the fn deploy command to build the Docker image, push the image to OCIR, and deploy the function to   Oracle Functions in your application.
  1.	From the Console UI, open the Cloud Shell.
  2.	Clone the Functions source code repository:
        git clone https://github.com/oracle/oracle-functions-samples.git
        ![image](https://user-images.githubusercontent.com/42166489/108231817-2a96de80-7168-11eb-8e5f-d9afa3c8d0cc.png)

  3.	Go to the samples/oci-load-file-into-adw-python folder
  cd oracle-functions-samples/samples/oci-load-file-into-adw-python
  4.	Deploy the function to the etl-app:
       fn -v deploy --app etl-app
      ![image](https://user-images.githubusercontent.com/42166489/108231857-34204680-7168-11eb-98f2-50539e9c6ae5.png)

 

    After you deploy the function, you need to set function configuration values so the function knows how to connect to the Autonomous Database.
        Using the Fn CLI, set the following configuration values. Make sure you replace the [ORDS_BASE_URL] and [DB_PASSWORD] with your values:
        fn config function etl-app oci-load-file-into-adw-python ords-base-url [ORDS_BASE_URL]
        fn config function etl-app oci-load-file-into-adw-python db-schema admin
        fn config function etl-app oci-load-file-into-adw-python db-user admin
        fn config function etl-app oci-load-file-into-adw-python dbpwd-cipher [DB-PASSWORD]
        fn config function etl-app oci-load-file-into-adw-python input-bucket input-bucket
        fn config function etl-app oci-load-file-into-adw-python processed-bucket processed-bucket
      ![image](https://user-images.githubusercontent.com/42166489/108232147-79dd0f00-7168-11eb-91a9-8411c43a2205.png)

 

## STEP 7: Create an Event rule
  In this step, you will configure a Cloud Event to trigger the function when you drop the files into the input-bucket.
  1.	From Console UI, open navigation and select Application Integration and click Events Service.
   <p><figure><img src="https://user-images.githubusercontent.com/42166489/108232201-87929480-7168-11eb-99a3-3cb7cccde652.png"></img></figure></p>
  2.	Select your development compartment from the Compartment list.
  3.	Click Create Rule.     <br/><p><figure><img src="https://user-images.githubusercontent.com/42166489/108232396-bd377d80-7168-11eb-97a3-bdd69e6c60e3.png"></img></figure></p>

  4.	For display name, enter load_CSV_into_ADW.
  5.	For description, enter Load CSV file into ADW.
  6.	Create three rules. You can click Another Condition to add more conditions
  7.	Under Actions, select Functions:
      o	For function compartment, select your development compartment.
      o	For function application, select etl-app.
      o	For function, select oci-load-file-into-adw-python.
 
  Click Create Rule.
  <p><figure><img src="https://user-images.githubusercontent.com/42166489/108232480-d7715b80-7168-11eb-89bc-31e93df48700.png"></img></figure></p>


  ## STEP 8: Test the function
  
  To test the function, you can upload a .csv file to the input-bucket. You can do that from the Console UI or the Cloud Shell using the OCI CLI.
  1.	Open the Cloud Shell.
  2.	Go to the functions folder: 

          cd ~/oracle-functions-samples/samples/oci-load-file-into-adw-python
  3.	 Use the OCI CLI to upload file1.csv to the input-bucket:

          $ oci os object put  --bucket-name input-bucket --file file1.csv
          Uploading object  [####################################]  100%
            {
              "etag": "607fd72d-a041-484c-9ee0-93b9f5488084",
              "last-modified": "Tue, 20 Oct 2020 18:03:50 GMT",
              "opc-content-md5": "O8mZv0X2gLagQGT5CutWsQ=="
            }
  ![image](https://user-images.githubusercontent.com/42166489/108232913-43ec5a80-7169-11eb-957d-10f88462b780.png)


  To see the data in the database, follow these steps:
  1. From the OCI console, navigate to Autonomous Data Warehouse.
  2. Select your development compartment from the Compartment list.
  3. Select Transaction Processing from the Workload Type list.
  4. Click on the database name (funcdb).
  5. Click the Service Console.
  6. Click Development link from the side bar.
  7. Click SQL Developer Web.
  8. Use ADMIN and the admin password to authenticate.
  9. In the worksheet, enter the following query:
  10. select UTL_RAW.CAST_TO_VARCHAR2( DBMS_LOB.SUBSTR( JSON_DOCUMENT, 4000, 1 )) AS json from regionsnumbers
  11. Click the green play button to execute the query.
  12. The data from the CSV file is in the Query Result tab.
 
![image](https://user-images.githubusercontent.com/42166489/108233013-5cf50b80-7169-11eb-94d4-25b0958862a3.png)



