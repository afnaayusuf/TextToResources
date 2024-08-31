# TextToResources
"""
This repository allows for the generation of PDFs, cheatsheets, and flashcards from text captions derived from audio files. Utilizing the LLaMA 3 LLM model, Lecturify processes the captions to create these educational resources and serves them on a local host.
"""

from langchain_ollama import OllamaLLM
from langchain_core.prompts import ChatPromptTemplate
from fpdf import FPDF
import markdown
import re
from fastapi import FastAPI, HTTPException
import uvicorn
from fastapi.responses import FileResponse, PlainTextResponse,JSONResponse
from pydantic import BaseModel
from fastapi.middleware.cors import CORSMiddleware

model = OllamaLLM(model='llama3')  

def pdf_generation(user_prompt):
    pdf_template = """
    Answer the question below:
    Here is the conversation history: {context}
    Question: {question}
    Above given is a lecture. Generate lecture notes of the lecture that is over 1000 words long. Ensure that these notes captures the significant details of the text in over 1000 words. Output this 1000 word notes using markdown formatting. it must be in markdown and structure into title, paragraph, lists.
    """
    pdf_input_prompt = {
    "context": "",  
    "question": f"{user_prompt}\n"
    }
    pdf_prompt = ChatPromptTemplate.from_template(template=pdf_template)
    pdf_chain = pdf_prompt | model
    pdf_result = pdf_chain.invoke(pdf_input_prompt)
    # print(pdf_result)
    pdf_formatted_result = str(pdf_result).replace("h4", "h3") # TODO Remove
    pdf_formatted_result = pdf_formatted_result.replace("→", "->")
    final_result = ""

    for line in pdf_formatted_result.split("\n"):
        if "--" not in line and "•" not in line:
            final_result += line
            final_result+='\n'
    return markdown.markdown(final_result)

def initialize():
    pdf = FPDF()
    pdf.set_auto_page_break(auto=True, margin=15)
    pdf.add_page()
    pdf.set_font("Times", size=14)
    return pdf

def html2pdf(pdf, html_text):
    pdf.write_html(html_text)

def create(html_text, output_file):
    pdf = initialize()
    html2pdf(pdf, html_text)
    pdf.output(output_file)

#create(pdf_generation(pdf_template), "summary.pdf")


def flashcard_generation(user_prompt):
    flashcard_template = """
    Answer the question below:
    Here is the conversation history: {context}
    Question: {question}
    Generate exactly two lists based on the given data. The first list should contain more than 15 questions that are logical, advanced, problem-solving, theoretical, and calculation-based. The second list should contain the corresponding answers to these questions. Provide only the two lists in the response, without any additional text, expressions, or conclusions.
    """

    flashcard_input_prompt = {
    "context": "",  
    "question": f"{user_prompt}\n"
    }
    flashcard_prompt = ChatPromptTemplate.from_template(template=flashcard_template)
    flashcard_chain = flashcard_prompt | model
    flashcard_result = flashcard_chain.invoke(flashcard_input_prompt)
    question_pattern = re.compile(r'\d+\.\s.*?\?')
    answer_pattern = re.compile(r'\d+\.\s.*?(?=\d+\.\s|\Z)', re.DOTALL)
    questions = question_pattern.findall(flashcard_result)
    answers = answer_pattern.findall(flashcard_result)
    answers = [answer.strip() for answer in answers]
    questions = [question.strip() for question in questions]
    def remove_elements(questions, answers):
        answers[:] = [item for item in answers if item not in questions] 
    remove_elements(questions, answers)
    def transfer_first_element(answers, questions):
        if answers:  
            element = answers.pop(0)  
            questions.append(element)
    transfer_first_element(answers, questions)
    def remove_substring_from_last_element(string_list, substring, substring_):
        if string_list:  
            last_element = string_list[-1]  
            if substring in last_element:  
                last_element = last_element.replace(substring, "")
                string_list[-1] = last_element
            elif substring_ in last_element:
                last_element = last_element.replace(substring_, "")
                string_list[-1] = last_element
    remove_substring_from_last_element(questions, "\n\nList 2: Answers", "\n\n*List 2: Answers*")
    list_len = len(questions)
    if list_len > 0:  
        questions.pop(list_len - 1)  
    
    flashcard_dict = dict(zip(questions, answers))
    return flashcard_dict




def cheatsheat_generation(user_prompt):
    cheatsheat_template = """
    Answer the question below:
    Here is the conversation history: {context}
    Question: {question}
    Generate 30 bullet points from the above given lecture.
    """

    cheatsheat_input_prompt = {
    "context": "",  
    "question": f"{user_prompt}\n"
    }
    pdf_prompt = ChatPromptTemplate.from_template(template=cheatsheat_template)
    pdf_chain = pdf_prompt | model
    pdf_result = pdf_chain.invoke(cheatsheat_input_prompt)
    cheatsheat_formatted_result = str(pdf_result).replace("h4", "h3") # TODO Remove
    cheatsheat_formatted_result = cheatsheat_formatted_result.replace("→", "->")
    cheatsheat_final_result = ""

    for line in cheatsheat_formatted_result.split("\n"):
        if "--" not in line and "•" not in line:
            cheatsheat_final_result += line
            cheatsheat_final_result+='\n'
    return markdown.markdown(cheatsheat_final_result)

#API
pdf_sum="summary.pdf"
pdf_cheat="cheatsheet.pdf"

app = FastAPI()
app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_methods=["*"],
    allow_headers=["*"],
    allow_credentials=True,
)

@app.get("/v1/info")
async def home():
    return {"200": "OK"}

@app.get("/v1/get_pdf/{data}", response_class=FileResponse)
async def get_pdf(data: str):
    create(pdf_generation(data), "summary.pdf")
    return FileResponse(pdf_sum, media_type='application/pdf', filename='summary.pdf')

"""@app.get("/text", response_class=JSONResponse)
async def get_text():
    return sample_text"""


"""@app.get("/title", response_class=JSONResponse)
async def get_title():
    return sample_title"""

@app.get("/v1/flashcard/{data}",response_class=JSONResponse)
async def flashc(data: str):
    a=flashcard_generation(data)
    return a

@app.get("/v1/cheats/{data}",response_class=JSONResponse)
async def cheatsh(data: str):
    create(cheatsheat_generation(data),"cheatsheet.pdf")
    return FileResponse(pdf_cheat,media_type='application/pdf',filename='cheatsheet.pdf')

"""f name == "main":
    uvicorn.run(app, host = "0.0.0.0", port = 8001, log_level = "debug")"""
