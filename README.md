# DSC-Vision-AI-Workshop

# Setup
1. Install Anaconda from [here](https://www.anaconda.com/products/individual) which provides a virtual environment to run your python scripts and libraries. **Make sure to add anaconda to your path.**
2. After downloading it, create an environment called 'visionAI' by running these commands:

```bash
conda create --name visionAI
```
```bash
activate visionAI
```
3. Once you're in the env, install the python libraries necessary for GCP to work:
```bash
pip install --upgrade google-cloud-storage
```
4. Follow the installation instructions [here](https://cloud.google.com/sdk/docs/install) to install the Cloud SDK.
5. **Make sure to add the SDK bin file to your path (environment variables).** It should be of this file path pattern assuming you installed it at the default location:  
C:\Users\\{your_name}\AppData\Local\Google\Cloud SDK\google-cloud-sdk\bin
6. Create a free account on GCP [here](https://cloud.google.com/) and navigate to your console.

# Tutorial
### Step 1 - Setting up the GCP console environment:
1. Create a new project called 'DSC Vision AI Workshop'.
2. **Make sure to have billing enabled.** Don't worry, the stuff we are doing is really low-level so it won't actually charge you money so just add any credit card. The free student account should also include $500 worth of free credits so there's no way you'll ever have to pay real money. 
3. Activate the following APIs by navigating to their respective API pages (you can search for them in the navbar):  
**Cloud Vision API  
Cloud Translation API  
Cloud Text-to-Speech API** 

### Step 2 - Setting up your local machine environment:

1. Make sure you are in the env that you created before called visionAI.
2. Install the following API libraries:
```bash
pip install --upgrade google-cloud-vision
```
```bash
pip install --upgrade google-cloud-translate
```
```bash
pip install --upgrade google-cloud-texttospeech
```
### Step 3 - Setting up permissions for using the translation API:

1. Click the navigation menu > IAM & Admin > Service Accounts > click Create Service Account
2. Put any name in **all lowercase** for name.
3. For 'Select a Role', filter by typing 'translation' and select 'Cloud Translation API Editor.
4. Click Done on the final page. 
5. Click the 3 dots under Actions and select Create key, then select JSON. This should download the key to your machine. 
6. In your terminal, set the GOOGLE_APPLICATION_CREDENTIALS variable using the following command. Replace path_to_key with the path to the downloaded JSON file containing your new service account key:
```bash
set GOOGLE_APPLICATION_CREDENTIALS=path_to_key
```


## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License
[MIT](https://choosealicense.com/licenses/mit/)
