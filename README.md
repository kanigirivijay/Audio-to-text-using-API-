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
