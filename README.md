# Python Geocoding Service SSE for Qlik
#### This is intended as an example capability for SSE and is not intended for production.


## REQUIREMENTS

- **Assuming prerequisite: [Python with Qlik Sense AAI – Environment Setup](https://s3.amazonaws.com/dpi-sse/DPI+-+Qlik+Sense+AAI+and+Python+Environment+Setup.pdf)**
    - This is not mandatory and is intended for those who are not as familiar with Python to setup a virtual environment. Feel free to follow the below instructions flexibly if you have experience.
- Qlik Sense February 2018+
- *Note: the Geocode() function may be used with QlikView as of November 2017+. Table loads (script tensor) are currently not supported in QlikView, but scalar functions may be used in both the script and front-end. Be aware of the performance implications here as scalar funtions are called record-by-record.*
    - *See how to setup Analytic Connections within QlikView [here](https://help.qlik.com/en-US/qlikview/November2017/Subsystems/Client/Content/Analytic_connections.htm)*
- Python 3.5.3 64 bit
- Python Libraries: grpcio, geopy
- Qlik GeoAnalytics (to visualize real-time geocoding from the front-end on a map)

## LAYOUT

- [Prepare your Project Directory](#prepare-your-project-directory)
- [Install Python Libraries and Required Software](#install-python-libraries-and-required-software)
- [Setup an AAI Connection in the QMC](#setup-an-aai-connection-in-the-qmc)
- [Copy the Package Contents and Import Examples](#copy-the-package-contents-and-import-examples)
- [Prepare And Start Services](#prepare-and-start-services)
- [Leverage Geocoding from within Qlik](#leverage-geocoding-from-within-qlik)
- *[*Add your own API Key for Google or Change Connection to http*](#add-your-own-api-key-for-google-or-change-connection-to-http)*

 
## PREPARE YOUR PROJECT DIRECTORY
>### <span style="color:red">ALERT</span>
><span style="color:red">
>Virtual environments are not necessary, but are frequently considered a best practice when handling multiple Python projects.
></span>

1. Open a command prompt
2. Make a new project folder called QlikSenseAAI, where all of our projects will live that leverage the QlikSenseAAI virtual environment that we’ve created. Let’s place it under ‘C:\Users\{Your Username}’. If you have already created this folder in another guide, simply skip this step.

3. We now want to leverage our virtual environment. If you are not already in your environment, enter it by executing:

```shell
$ workon QlikSenseAAI
```

4. Now, ensuring you are in the ‘QlikSenseAAI’ folder that you created (if you have followed another guide, it might redirect you to a prior working directory if you've set a default, execute the following commands to create and navigate into your project’s folder structure:
```
$ cd QlikSenseAAI
$ mkdir Geocoding
$ cd Geocoding
```


5. Optionally, you can bind the current working directory as the virtual environment’s default. Execute (Note the period!):
```shell
$ setprojectdir .
```
6. We have now set the stage for our environment. To navigate back into this project in the future, simply execute:
```shell
$ workon QlikSenseAAI
```

This will take you back into the environment with the default directory that we set above. To change the
directory for future projects within the same environment, change your directory to the desired path and reset
the working directory with ‘setprojectdir .’


## INSTALL PYTHON LIBRARIES AND REQUIRED SOFTWARE

1. Open a command prompt or continue in your current command prompt, ensuring that you are currently within the virtual environment—you will see (QlikSenseAAI) preceding the directory if so. If you are not, execute:
```shell
$ workon QlikSenseAAI
```
2. Execute the following commands. If you have followed a previous guide, you have more than likely already installed grpcio):

```shell
$ pip install grpcio
$ pip install geopy
```

## SET UP AN AAI CONNECTION IN THE QMC

1. Navigate to the QMC and select ‘Analytic connections’
2. Fill in the **Name**, **Host**, and **Port** parameters -- these are mandatory.
    - **Name** is the alias for the analytic connection. For the example qvf to work without modifications, name it 'PythonGeocoding'
    - **Host** is the location of where the service is running. If you installed this locally, you can use 'localhost'
    - **Port** is the target port in which the service is running. This module is setup to run on 50095, however that can be easily modified by searching for ‘-port’ in the ‘\_\_main\_\_.py’ file and changing the ‘default’ parameter to an available port.
3. Click ‘Apply’, and you’ve now created a new analytics connection.


## COPY THE PACKAGE CONTENTS AND IMPORT EXAMPLES

1. Now we want to setup our geocoding service and app. Let’s start by copying over the contents of the example
    from this package to the ‘..\QlikSenseAAI\Geocoding\’ location. Alternatively you can simply clone the repository.
2. After copying over the contents, go ahead and import the example qvf found [here](https://s3.amazonaws.com/dpi-sse/qlik-python-sse-geocoding/DPI+-+Python+Geocoding.qvf).
3. Lastly, import the qsvariable extension zip file found [here](https://github.com/erikwett/qsVariable) if you are using Qlik Sense.


## PREPARE AND START SERVICES

1. At this point the setup is complete, and we now need to start the geocoding extension service. To do so, navigate back to the command prompt. Please make sure that you are inside of the virtual environment.
2. Once at the command prompt and within your environment, execute (note two underscores on each side):
```shell
$ python __main__.py
```
3. We now need to restart the Qlik Sense engine service so that it can register the new AAI service. To do so,
    navigate to windows Services and restart the ‘Qlik Sense Engine Service’
4. You should now see in the command prompt that the Qlik Sense Engine has registered the functions *Geocode()* and *GeocodeScript()* from the extension service over port 50095, or whichever port you’ve chosen to leverage.


## LEVERAGE GEOCODING FROM WITHIN QLIK

1. The *Geocode()* function leverages the [geopy package](https://geopy.readthedocs.io/en/stable/) and accepts two mandatory arguments:
    - *Text (string)*: the full address or named location
    - *API Selection (string)*: this is what type of data you'd like to return and can be:
        - *Google* - recommended, limited to 2500 per day. You can extend with an API Key. See below.
        - *Open Street*
2. Example function calls:

	*Returns the geocoded coordinates of Monks Cafe in Philadelphia*
    
    ``` PythonGeocoding.Geocode('Monks Cafe','Google') ``` 
    
    *Returns the geocoded coordinates of Qlik's Office in NY*
    
    ``` PythonGeocoding.Geocode('292 Madison Ave, New York, NY 10017','Google') ``` 
    
3. There is another script function exposed called *GeocodeScript()*. This function can only be used in the script and is leveraged via the [**LOAD ... EXTENSION ...**](https://help.qlik.com/en-US/sense/February2018/Subsystems/Hub/Content/Scripting/ScriptRegularStatements/Load.htm) mechanism which was added as of the February 2018 release of Qlik Sense. This function takes three fields:
       
       address, id (numeric), api

See the below example of how to utilize the *GeocodeScript()* function in the load script:

*Note you can also use the *Geocode()* function in the script like any other native function, but it will operate record by record (*Scalar*) vs in one call (*Tensor*) as seen below. If you are using QlikView, at this point in time you must use the *Geocode()* function like any other native Qlik function in the script, and cannot use the tensor method. Both examples below:

```
Addresses:
FIRST 500
LOAD
    "Store Number" AS ID,
    Name,
    "Features - Service",
    "Street Address" & ', ' & City & ', ' & State & ', ' & Zip & ' ' & Country AS Address,
    'Google' AS API
FROM [lib://Builds (qlik_qservice)/Starbucks\All_Starbucks_Locations_in_the_US_-_Map.csv.txt]
(txt, codepage is 28591, embedded labels, delimiter is ',', msq);

// SCRIPT TENSOR - CURRENTLY QLIK SENSE ONLY
Geocodes:
LOAD
	Field1 AS ID,
    Field2 AS Geocode
EXTENSION PythonGeocoding.GeocodeScript(Addresses{Address, ID, API});

STORE Geocodes INTO [lib://Builds (qlik_qservice)/Starbucks\MinedGeocodes.qvd](qvd);



// SCALAR SOLUTION - CAN BE USED IN QLIKVIEW NOV 2017+ BUT NOT AS EFFICIENT AS ABOVE
// Addresses:
// FIRST 500
// LOAD
//     "Store Number" AS ID,
//     Name,
//     "Features - Service",
//     PythonGeocoding.Geocode("Street Address" & ', ' & City & ', ' & State & ', ' & Zip & ' ' & Country,'Google') AS Geocode
// FROM [lib://Builds (qlik_qservice)/Starbucks\All_Starbucks_Locations_in_the_US_-_Map.csv.txt]
// (txt, codepage is 28591, embedded labels, delimiter is ',', msq);
```

## Add your own API Key for Google or Change Connection to http
1. If you hit the 2500 record daily limit imposed by the Google Geocoding API, you can [add your own key by following these steps](https://developers.google.com/maps/documentation/geocoding/get-api-key). To add this key to the SSE, open up ‘\_\_main\_\_.py’ and search for ```geolocator = GoogleV3()``` (note this exists in both functions so you'd edit both occurrences). Change this line to ```geolocator = GoogleV3(api_key='YOUR_API_KEY_HERE')```
2. If you are hitting other geocoding query errors, if you do not have an API key, try changing ```geolocator = GoogleV3()``` to ```geolocator = GoogleV3(scheme='http')``` in the ‘\_\_main\_\_.py’ file (note this exists in both functions so you'd edit both occurrences).