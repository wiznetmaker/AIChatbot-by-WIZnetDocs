# PDF QA Chatbot Project

## Introduction
This project involves the development of a PDF QA chatbot utilizing the Retrieval-Augmented Generation (RAG) approach, designed for corporate environments to create document-based Q&A systems. It combines Azure OCR, ChromaDB, and the OpenAI API to create an advanced document-based question-answering system.
![image](https://github.com/dbtjr1103/chat-pdf/assets/115054808/deacda07-39f0-47cf-a712-cad9eb9c7c95)
![image](https://github.com/dbtjr1103/chat-pdf/assets/115054808/dde9a21c-549c-4bec-a5b9-08df34ec1b3b)

## What is RAG?
RAG, or Retrieval Augmented Generation, is a technique in natural language processing to generate customized outputs beyond the scope of existing training data. It consists of two main components: a retriever and a generator. The retriever finds the most relevant information in the knowledge base related to a given input, and the generator creates consistent, relevant responses based on these search results.
![image](https://github.com/dbtjr1103/chat-pdf/assets/115054808/6375adec-0150-4588-9181-0e92639bdb75)

## Project Setup and Implementation
We install necessary tools and set up a chatbot UI using Streamlit.

```bash
pip install streamlit azure-ai-formrecognizer openai chromadb tabulate
```

## Extracting and Vectorizing Text from PDF
Texts are extracted from PDFs using Azure AI document intelligence and then vectorized for storage in ChromaDB.
![image](https://github.com/dbtjr1103/chat-pdf/assets/115054808/e0f347c4-3adc-46da-b185-cea9fa429540)

```python
# Create a temporary file to write the bytes to
with open("temp_pdf_file.pdf", "wb") as temp_file:
    temp_file.write(uploaded_file.read())

AZURE_COGNITIVE_ENDPOINT = "own_key"
AZURE_API_KEY = "own_key"
credential = AzureKeyCredential(AZURE_API_KEY)
AZURE_DOCUMENT_ANALYSIS_CLIENT = DocumentAnalysisClient(AZURE_COGNITIVE_ENDPOINT, credential)

# Open the temporary file in binary read mode and pass it to Azure
with open("temp_pdf_file.pdf", "rb") as f:
    poller = AZURE_DOCUMENT_ANALYSIS_CLIENT.begin_analyze_document("prebuilt-document", document=f)
    doc_info = poller.result().to_dict()
```

## Generating Chatbot Responses
The chatbot queries ChromaDB based on user input to fetch related data and uses OpenAI's GPT-4 model to generate appropriate responses.
![image](https://github.com/dbtjr1103/chat-pdf/assets/115054808/3b8b2803-d197-4b7f-a07c-b5195398d66d)

```python
# query ChromaDB based on your prompt, taking the top 5 most relevant result. These results are ordered by similarity.
q = collection.query(
    query_texts=[prompt],
    n_results=3,
)

results = q["documents"][0]

prompts = []
for r in results:
    # construct prompts based on the retrieved text chunks in results 
    prompt = "Please extract the following: " + prompt + "  solely based on the text below. Use an unbiased and journalistic tone. If you're unsure of the answer, say you cannot find the answer. \n\n" + r

    prompts.append(prompt)
prompts.reverse()

openai_res = openai.ChatCompletion.create(
    model="gpt-4",
    messages=[{"role": "assistant", "content": prompt} for prompt in prompts],
    temperature=0,
)

response = openai_res["choices"][0]["message"]["content"]
```

## Running and Testing the Streamlit App
The chatbot is tested in real-time to optimize the user experience.

```python
st.write("# Chat with WIZnet Docs")

uploaded_file = st.file_uploader("Choose a PDF file", type="pdf")

# Display chat messages from history on app rerun
for message in st.session_state.messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

if prompt := st.chat_input("What do you want to say to your PDF?"):
    # ... (User input processing and chatbot response generation code)

    # append the response to chat history
    st.session_state.messages.append({"role": "assistant", "content": response})
```

![image](https://github.com/dbtjr1103/chat-pdf/assets/115054808/30e52950-8a75-4d56-bb75-fd75f2221d54)

## Conclusion and Outlook
We've learned how to efficiently build a PDF QA chatbot by combining Azure OCR, ChromaDB, and OpenAI API. This method is expected to be adopted across various industries, serving as a powerful tool for document-based Q&A systems.

Source code link: [WIZnet maker](https://maker.wiznet.io/Benjamin/projects/how%2Dto%2Dbuild%2Dthe%2Dwiznet%2Dai%2Dchatbot%2Dwith%2Dazure%2Dopenai%2Dand%2Dchromadb/)

Thank you.
