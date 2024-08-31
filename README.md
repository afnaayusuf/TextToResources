# TextToResources
<br>

TextToResources is a powerful tool that generates PDFs, cheatsheets, and flashcards from text captions derived from audio files. Leveraging the LLaMA 3 LLM model, this repository processes captions to create valuable educational resources, which are then served on a local host for easy access.

<br>

# Project Background
<br>

This project was developed by a team of three SRMIST students—Afnaan, Jeswyn, and Hashin—as part of the Red Bull Basement contest. Our innovative approach to converting text captions into versatile learning materials earned us a spot as intra-university finalists. The project garnered significant interest from judges, who saw great potential for further development. Unfortunately, due to time constraints, the interfacing wasn't fully completed, which sadly prevented us from being fast-tracked to the next tier of the competition.

<br>

# Features
<br>

PDF Generation: Converts text captions into well-formatted PDF documents for easy reading and printing.
Cheatsheet Creation: Summarizes key points from the text captions into concise cheatsheets.
Flashcard Development: Transforms text captions into interactive flashcards for effective study and revision.

<br>

# How It Works
<br>

Upload Captions: Begin by uploading text captions derived from an audio file.
Processing: The LLaMA 3 LLM model processes the uploaded captions.
Output Generation: Generate PDFs, cheatsheets, and flashcards.
Local Host Access: Access all generated resources on a local host.

<br>

## 1. Install the LLaMA 3 Model (4B)
<br>
Before running the application, ensure that the LLaMA 3 LLM model (4B) is properly installed:
Download the LLaMA 3 model (4B) from the official repository or source.
Follow the installation instructions provided by the LLaMA model's maintainers.
Ensure that all dependencies required for LLaMA 3 are installed and configured correctly on your system.
<br>

## 2. Installation and Setup

>  Clone the repository and install the necessary dependencies:
<br><br>

```bash
Copy code
git clone https://github.com/your-username/TextToResources.git
cd TextToResources
pip install -r requirements.txt
```
<br>

## 3. Running the Application

>  After installing the model and setting up the environment, run the application:
<br><br>

```bash
Copy code
python app.py
```
<br>

## 4. Generate Educational Resources
<br>

* Upload Captions: Begin by uploading text captions derived from an audio file.

* Processing: The LLaMA 3 LLM model processes the uploaded captions to generate educational resources.

* Access Outputs: Access the generated PDFs, cheatsheets, and flashcards on the local host.

* Access the Local Host

* Open your web browser and navigate to http://localhost:8000 to view and download the generated resources.
<br>

## 5. Future Development

<br>

We plan to enhance the interfacing and improve the functionality of the tool based on the feedback received. We are open to collaborations and suggestions for further development!
