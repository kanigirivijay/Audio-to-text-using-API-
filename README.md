This project is a Streamlit-based web application that leverages the power of the OpenAI Whisper model to translate audio from any language into English text. It provides a clean, interactive interface for users to upload audio files, perform the translation, and download the results.

Key Features
Interactive Web Interface: A user-friendly, responsive interface built with Streamlit, featuring a sidebar for controls and a main area for results.

Whisper Model Integration: Utilizes the Whisper model for highly accurate audio-to-text transcription and translation.

Dynamic Model Loading: Allows users to select and load different sizes of the Whisper model directly from the interface, with the model being stored in st.session_state to prevent re-loading on every user interaction.

Audio File Upload: Supports popular audio formats including WAV, MP3, and M4A via a drag-and-drop uploader.

Text-to-Speech: Uses the gTTS library to convert the translated English text back into an audio file.

Downloadable Output: Provides buttons to download both the translated audio file (.mp3) and the translated text (.txt).

Step-by-Step Methodology
Here are the key steps involved in the application's design and why they were chosen:

1. Setup and Initialization
Streamlit & Libraries: The app begins by importing streamlit to build the UI, whisper for the core AI task, and gTTS for text-to-speech functionality. Other libraries like tempfile and os are imported for file handling.

Session State: We use st.session_state to maintain the state of the application across user interactions. This is crucial for:

model: Storing the loaded Whisper model so it doesn't need to be reloaded every time a button is clicked.

model_loaded: A boolean flag to track if the model is ready for use.

uploaded_audio_path: To remember the path of the user's uploaded audio file.

2. User Interface (UI) Design
Sidebar for Controls: A sidebar is used to keep the main actions—model selection, audio upload, and translation buttons—organized and separate from the output, improving user experience.

Dynamic Components:

A selectbox is used to let the user choose the Whisper model size (tiny, base, small, etc.).

A file_uploader widget handles the audio file upload, which automatically returns the file data.

st.button triggers the computationally intensive tasks of loading the model and translating the audio.

3. Audio Handling and Translation
File Handling: When an audio file is uploaded, the save_uploaded_audio function is called. It uses tempfile.NamedTemporaryFile to save the file to a temporary location. This is a secure and clean way to handle uploaded files without cluttering the project directory.

Translation Process:

The if st.sidebar.button("Translate Audio") block kicks off the main process.

A st.spinner is used to provide visual feedback to the user, indicating that the translation is in progress.

st.session_state.model.transcribe(audio_path, task="translate") is the core of the translation. The task="translate" parameter instructs the Whisper model to translate the audio to English, regardless of the source language.

The transcribe function returns both the translated text and the detected source language code, which are then displayed to the user.

4. Output and Download Options
Displaying Results: The application uses st.markdown to display the translated text and detected language in a formatted, readable way.

Text-to-Speech: The gTTS library is used to create an audio version of the translated English text. This enhances the user experience by allowing them to both read and listen to the translation.

Download Buttons: st.sidebar.download_button is used to create buttons that automatically package and download the translated_audio.mp3 and translated_text.txt files for the user.

How to Run the Project
Clone the repository:

Bash

git clone [your-repository-url]
cd [your-repository-name]
Install the required libraries:

Bash

pip install streamlit openai-whisper gTTS
Note: For Whisper, you may also need to install ffmpeg. On macOS, you can use brew install ffmpeg. On Windows, you can download it and add it to your system's PATH.

Run the Streamlit application:

Bash

streamlit run [your_script_name].py








here si the code 



import streamlit as st
import whisper
import tempfile
import os
import logging
from gtts import gTTS

# Set up logging
logging.basicConfig(level=logging.DEBUG)

# Define the folder path for saving audio files
SAVE_DIR = r"C:\Users\user\Downloads\project_audio_files"
os.makedirs(SAVE_DIR, exist_ok=True)

# Initialize session state variables
if 'model' not in st.session_state:
    st.session_state.model = None
if 'model_loaded' not in st.session_state:
    st.session_state.model_loaded = False
if 'uploaded_audio_path' not in st.session_state:
    st.session_state.uploaded_audio_path = None
if 'is_loading' not in st.session_state:
    st.session_state.is_loading = False

# Function to load the Whisper model
def load_model(model_name):
    try:
        st.session_state.model = whisper.load_model(model_name)
        st.session_state.model_loaded = True
        st.sidebar.success(f"Whisper Model '{model_name}' Loaded")
    except Exception as e:
        st.sidebar.error(f"Error loading model: {e}")
        logging.error(f"Model loading error: {e}")

# Function to save uploaded audio
def save_uploaded_audio(uploaded_file):
    try:
        with tempfile.NamedTemporaryFile(delete=False, suffix=".wav") as temp_audio_file:
            temp_audio_file.write(uploaded_file.read())
            st.session_state.uploaded_audio_path = temp_audio_file.name
        return st.session_state.uploaded_audio_path
    except Exception as e:
        st.sidebar.error(f"Error saving audio file: {e}")
        logging.error(f"Audio save error: {e}")

# Function to map detected language codes to full language names
def language_code_to_name(code):
    code_map = {
        "en": "English", "hi": "Hindi", "fr": "French", "te": "Telugu",
        "es": "Spanish", "de": "German", "ja": "Japanese", "ta": "Tamil",
        "bn": "Bengali", "mr": "Marathi", "gu": "Gujarati", "pa": "Punjabi",
        "ml": "Malayalam", "or": "Odia", "ur": "Urdu", "pt": "Portuguese"
    }
    return code_map.get(code, "Unknown Language")

# Main Content: Welcome Image, Title, and Instructions
st.markdown("""
# Convert All Languages to English
Heyyy, upload your audio, get it in the form of text.
""", unsafe_allow_html=True)

# Sidebar: Model selection and audio upload
st.sidebar.header("Model Selection")
model_options = ["tiny", "base", "small", "medium", "large"]
model_name = st.sidebar.selectbox("Select Whisper Model", model_options)

# Load model button
if st.sidebar.button("Load Model"):
    load_model(model_name)

# Sidebar for uploading pre-recorded audio
st.sidebar.header("Upload Audio")
audio_file = st.file_uploader("Upload Audio", type=["wav", "mp3", "m4a"])

if audio_file:
    uploaded_audio_path = save_uploaded_audio(audio_file)
    if uploaded_audio_path:
        st.sidebar.success("Audio file uploaded and saved.")

# Play original audio playback
if st.session_state.uploaded_audio_path:
    st.sidebar.audio(st.session_state.uploaded_audio_path)

# Translation section
if st.session_state.model_loaded and st.session_state.uploaded_audio_path:
    st.sidebar.header("Translate Audio")
    if st.sidebar.button("Translate Audio"):
        st.session_state.is_loading = True
        with st.spinner('Translating audio...'):
            try:
                # Transcribe and detect language
                result = st.session_state.model.transcribe(st.session_state.uploaded_audio_path, task="translate")
                transcribed_text = result["text"]
                detected_language_code = result["language"]
                
                # Log the transcribed text for debugging
                logging.debug(f"Transcribed Text: {transcribed_text}")
                
                # Convert language code to full name
                detected_language_name = language_code_to_name(detected_language_code)
                
                # Display detected language and translated text
                st.sidebar.success("Translation complete")
                st.markdown(f"""
                ## Translated Text:
                {transcribed_text}
                """, unsafe_allow_html=True)
                st.markdown(f"""
                ## Detected Language:
                {detected_language_name}
                """, unsafe_allow_html=True)
                
                # Convert the translated text to English speech and save it
                output_audio_path = os.path.join(SAVE_DIR, "translated_audio.mp3")
                tts = gTTS(text=transcribed_text, lang="en")
                tts.save(output_audio_path)
                
                # Display the title above the AI voice player
                st.markdown('## Echo of AI: Hear It in English', unsafe_allow_html=True)
                
                # Play the translated audio in the Streamlit app
                with open(output_audio_path, "rb") as audio_file:
                    audio_bytes = audio_file.read()
                    st.audio(audio_bytes, format="audio/mp3")
                
                # Download options
                st.sidebar.download_button(
                    label="Download Translated Audio",
                    data=open(output_audio_path, "rb").read(),
                    file_name="translated_audio.mp3",
                    mime="audio/mp3"
                )
                
                # Download translated text option
                st.sidebar.download_button(
                    label="Download Translated Text",
                    data=transcribed_text,
                    file_name="translated_text.txt",
                    mime="text/plain"
                )
            except Exception as e:
                st.sidebar.error(f"Error translating audio: {e}")
                logging.error(f"Translation error: {e}")
            finally:
                st.session_state.is_loading = False















