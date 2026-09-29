# this is a simple RAG which is created for small business 
### most of the business needs a model which is completely oriented for there database here's the simple RAG 
### first import module
```bash
import os
import glob
from dotenv import load_dotenv
from pathlib import Path #for path detection inside our file 
import gradio as gr # interface to chat , framework
from openai import OpenAI
```
### load api key and model 
```bash
# Setting up

GEMINI_BASE_URL = "https://generativelanguage.googleapis.com/v1beta/openai/" # url to connect endpoints of gemini to openai
load_dotenv(override=True) # load dotenv file
openai_api_key = os.getenv('OPENAI_API_KEY') #get env file and load the api key
if openai_api_key:
    print(f"OpenAI API Key exists and begins {openai_api_key[:8]}")
else:
    print("OpenAI API Key not set")



openai = OpenAI(base_url=GEMINI_BASE_URL, api_key=openai_api_key) #create object of openai 
MODEL = "gemini-3.5-flash-lite"
```
### read employee data from your system to dictonary
```bash
knowledge = {}

filenames = glob.glob("knowledge-base/employees/*") # variable having files from given path

for filename in filenames:
    name = Path(filename).stem.split(' ')[-1] 
    with open(filename, "r", encoding="utf-8") as f: #open file and load the name in lowercase using function f.read
        knowledge[name.lower()] = f.read()
```
### check
```bash
knowledge
knowledge["lancaster"] #specific name
```
### using same load keys of product from file
```bash
filenames = glob.glob("knowledge-base/products/*")

for filename in filenames:
    name = Path(filename).stem
    with open(filename, "r", encoding="utf-8") as f:
        knowledge[name.lower()] = f.read()
```
### check
```bash
knowledge.keys()
```
### give system prompt , what you actually are whats your role kinda
```bash
SYSTEM_PREFIX = """
You represent Insurellm, the Insurance Tech company.
You are an expert in answering questions about Insurellm; its employees and its products.
You are provided with additional context that might be relevant to the user's question.
Give brief, accurate answers. If you don't know the answer, say so.

Relevant context:
```
### function to take message as input and take only alphabet and space to return relevant context
```bash
def get_relevant_context_simple(message):
    text = ''.join(ch for ch in message if ch.isalpha() or ch.isspace())
    words = text.lower().split()
    relevant_context = []
    for word in words:
        if word in knowledge:
            relevant_context.append(knowledge[word])
    return relevant_context
```
### another approch
```bash
def get_relevant_context(message):
    text = ''.join(ch for ch in message if ch.isalpha() or ch.isspace())
    words = text.lower().split()
    return [knowledge[word] for word in words if word in knowledge]
```
### check
```bash
get_relevant_context("Who is lancaster?")
get_relevant_context("Who is Lancaster and what is carllm?")
```
### this function doesn't give message if it can find relevant message
so a function for not found 
```bash
def additional_context(message):
    relevant_context = get_relevant_context(message)
    if not relevant_context:
        result = "There is no additional context relevant to the user's question."
    else:
        result = "The following additional context might be relevant in answering the user's question:\n\n"
        result += "\n\n".join(relevant_context)
    return result
```
```bash
print(additional_context("Who is Alex Lancaster?"))
```
### this thing don't have memory 
so we create a function which gets execute again and again for previous message
```bash
def chat(message, history):
    system_message = SYSTEM_PREFIX + additional_context(message)
    messages = [{"role": "system", "content": system_message}] + history + [{"role": "user", "content": message}]
    response = openai.chat.completions.create(model=MODEL, messages=messages)
    return response.choices[0].message.content
```
### gradio interface 
```bash
view = gr.ChatInterface(chat, type="messages").launch(inbrowser=True,debug=True)
```






