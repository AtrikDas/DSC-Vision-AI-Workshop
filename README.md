# DSC-Vision-AI-Workshop

# Introduction
Have you ever wondered if memes are funnier in French rather than English? In this workshop we will be learning how to translate memes from different languages into English. We will use the Vision API, Translation API, and the Text-to-speech API of GCP to achieve our objectives:
1. Pass text recognized by the Cloud Vision API to the Cloud Translation API.
2. Create and use Cloud Translation glossaries to personalize Cloud Translation API translations.
3. Create an audio representation of translated text using the Text-to-Speech API.

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

### Step 4 - Actual coding!
1. Clone this repository to get the skeleton of our project into your machine.
```bash
git clone https://github.com/AtrikDas/DSC-Vision-AI-Workshop.git
```
2. Open hybrid_tutorials.py and import the relevant libraries.
```python
import html
import io
import os

# Imports the Google Cloud client libraries
from google.api_core.exceptions import AlreadyExists
from google.cloud import texttospeech
from google.cloud import translate_v3beta1 as translate
from google.cloud import vision
```
3. Set up the env variables.
```python
# Setting up OS env variables
os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = r'{insert your own json key}'
PROJECT_ID = os.environ["GOOGLE_CLOUD_PROJECT"]
```
4. Use the Vision API to detect and extract text from an image.
```python
# Image detection code
def pic_to_text(infile):
    """Detects text in an image file

    ARGS
    infile: path to image file

    RETURNS
    String of text detected in image
    """

    # Instantiates a client
    client = vision.ImageAnnotatorClient()

    # Opens the input image file
    with io.open(infile, "rb") as image_file:
        content = image_file.read()

    image = vision.Image(content=content)

    # For dense text, use document_text_detection
    # For less dense text, use text_detection
    response = client.document_text_detection(image=image)
    text = response.full_text_annotation.text
    print("Detected text: {}".format(text))

    return text
```
5. Code to create glossary used for translation.
```python
# Create glossary code
def create_glossary(languages, project_id, glossary_name, glossary_uri):
    """Creates a GCP glossary resource
    Assumes you've already manually uploaded a glossary to Cloud Storage

    ARGS
    languages: list of languages in the glossary
    project_id: GCP project id
    glossary_name: name you want to give this glossary resource
    glossary_uri: the uri of the glossary you uploaded to Cloud Storage

    RETURNS
    nothing
    """

    # Instantiates a client
    client = translate.TranslationServiceClient()

    # Designates the data center location that you want to use
    location = "us-central1"

    # Set glossary resource name
    name = client.glossary_path(project_id, location, glossary_name)

    # Set language codes
    language_codes_set = translate.types.Glossary.LanguageCodesSet(
        language_codes=languages
    )

    gcs_source = translate.types.GcsSource(input_uri=glossary_uri)

    input_config = translate.types.GlossaryInputConfig(gcs_source=gcs_source)

    # Set glossary resource information
    glossary = translate.types.Glossary(
        name=name, language_codes_set=language_codes_set, input_config=input_config
    )

    parent = f"projects/{project_id}/locations/{location}"

    # Create glossary resource
    # Handle exception for case in which a glossary
    #  with glossary_name already exists
    try:
        operation = client.create_glossary(parent=parent, glossary=glossary)
        operation.result(timeout=90)
        print("Created glossary " + glossary_name + ".")
    except AlreadyExists:
        print(
            "The glossary "
            + glossary_name
            + " already exists. No new glossary was created."
        )    
```
6. Use the Translation API to translate the foreign language to English using the glossary we created earlier.
```python
# Translation code
def translate_text(
    text, source_language_code, target_language_code, project_id, glossary_name
):
    """Translates text to a given language using a glossary

    ARGS
    text: String of text to translate
    source_language_code: language of input text
    target_language_code: language of output text
    project_id: GCP project id
    glossary_name: name you gave your project's glossary
        resource when you created it

    RETURNS
    String of translated text
    """

    # Instantiates a client
    client = translate.TranslationServiceClient()

    # Designates the data center location that you want to use
    location = "us-central1"

    glossary = client.glossary_path(project_id, location, glossary_name)

    glossary_config = translate.types.TranslateTextGlossaryConfig(glossary=glossary)

    parent = f"projects/{project_id}/locations/{location}"

    result = client.translate_text(
        request={
            "parent": parent,
            "contents": [text],
            "mime_type": "text/plain",  # mime types: text/plain, text/html
            "source_language_code": source_language_code,
            "target_language_code": target_language_code,
            "glossary_config": glossary_config,
        }
    )

    # Extract translated text from API response
    return result.glossary_translations[0].translated_text   
```
7. Use the Text-to-speech API to create synthetic audio of your translated text.
```python
# Text-to-speech code
def text_to_speech(text, outfile):
    """Converts plaintext to SSML and
    generates synthetic audio from SSML

    ARGS
    text: text to synthesize
    outfile: filename to use to store synthetic audio

    RETURNS
    nothing
    """

    # Replace special characters with HTML Ampersand Character Codes
    # These Codes prevent the API from confusing text with
    # SSML commands
    # For example, '<' --> '&lt;' and '&' --> '&amp;'
    escaped_lines = html.escape(text)

    # Convert plaintext to SSML in order to wait two seconds
    #   between each line in synthetic speech
    ssml = "<speak>{}</speak>".format(
        escaped_lines.replace("\n", '\n<break time="2s"/>')
    )

    # Instantiates a client
    client = texttospeech.TextToSpeechClient()

    # Sets the text input to be synthesized
    synthesis_input = texttospeech.SynthesisInput(ssml=ssml)

    # Builds the voice request, selects the language code ("en-US") and
    # the SSML voice gender ("MALE")
    voice = texttospeech.VoiceSelectionParams(
        language_code="en-US", ssml_gender=texttospeech.SsmlVoiceGender.MALE
    )

    # Selects the type of audio file to return
    audio_config = texttospeech.AudioConfig(
        audio_encoding=texttospeech.AudioEncoding.MP3
    )

    # Performs the text-to-speech request on the text input with the selected
    # voice parameters and audio file type

    request = texttospeech.SynthesizeSpeechRequest(
        input=synthesis_input, voice=voice, audio_config=audio_config
    )

    response = client.synthesize_speech(request=request)

    # Writes the synthetic audio to the output file.
    with open(outfile, "wb") as out:
        out.write(response.audio_content)
        print("Audio content written to file " + outfile) 
```
8. The main code which runs the previous functions.
```python
def main():

    # Photo from which to extract text
    infile = "resources/example.png"
    # Name of file that will hold synthetic speech
    outfile = "resources/example.mp3"

    # Defines the languages in the glossary
    # This list must match the languages in the glossary
    #   Here, the glossary includes French and English
    glossary_langs = ["fr", "en"]
    # Name that will be assigned to your project's glossary resource
    glossary_name = "meme-glossary"
    # uri of .csv file uploaded to Cloud Storage
    glossary_uri = "gs://meme-bucket-3226/meme-glossary.csv"

    create_glossary(glossary_langs, PROJECT_ID, glossary_name, glossary_uri)

    # photo -> detected text
    text_to_translate = pic_to_text(infile)
    # detected text -> translated text
    text_to_speak = translate_text(
        text_to_translate, "fr", "en", PROJECT_ID, glossary_name
    )
    # translated text -> synthetic audio
    text_to_speech(text_to_speak, outfile)
```
### Step 5 - Running the code!
1. Set your project-id in the terminal:
```bash
set GOOGLE_CLOUD_PROJECT={your project-id}
```
2. Make sure the infile and outfile paths are correctly set and just run the code:
```bash
python hybrid_tutorial.py
```
If everything works fine you should find the correct files outputted in the resources folder!

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License
[MIT](https://choosealicense.com/licenses/mit/)
