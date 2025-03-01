# Chatbot powered by Groq hosted on Streamlit

In this tutorial, I'll build and deploy a personalised AI-powered chat application using Streamlit and the latest AI model llama-3.1-8b-instant. We'll use Groq for faster inference. Also I am going to **deploy it for free!**

## Getting Started

First sign in to [https://groq.com/](https://groq.com/) and click `start building`
![Groq API](./images/1.png)

Click on `Create API key` then create a new key, copy it and keep it somewhere safe.

![Groq API Key](./images/2.png)

Now install the necessary libraries:
Create the requirements.txt file and paste this

```txt
groq
streamlit
python-dotenv
```

Install these using

```bash
pip install -r requirements.txt
```

Let's create our `main.py` file and import the required libraries:

```python
import os
from dotenv import dotenv_values
import streamlit as st
from groq import Groq
```

We'll use streamlit for building the chat interface, dotenv for handling environment variables, and groq for fast inference from the AI model.

## Configuring the Page

Let's set up the page configuration using Streamlit:

```python
st.set_page_config(
    page_title="The Tech Buddy ",
    page_icon="",
    layout="centered",
)
```

This will give our chat application a professional look and feel.

## Handling Environment Variables

We'll use environment variables to store sensitive information like API keys and and the application specific prompts.
In your root folder create a `.env` file like this:

```env
GROQ_API_KEY='YOUR_GROQ_API_KEY'

INITIAL_RESPONSE="Hey there! Welcome to the hackathon! I'm here to help you with anything coding-related or brainstorming your next big idea. What's your project about?"

CHAT_CONTEXT="You are a tech enthusiast participating in a hackathon, and your job is to help other developers by providing coding advice, brainstorming ideas, and offering guidance on various hackathon challenges. You speak in a friendly, team-oriented manner and are always ready to give useful tips."

INITIAL_MSG="Hello! I'm here to help you with anything you need during the hackathon—whether it's coding challenges, project ideas, or even troubleshooting. Let's build something awesome!"
```

This part is crucial to personalize your application as per your need. So play with it and explore.

Now configure this environment variables in our python file:

```python
try:
    secrets = dotenv_values(".env")  # for dev env
    GROQ_API_KEY = secrets["GROQ_API_KEY"]
except:
    secrets = st.secrets  # for streamlit deployment
    GROQ_API_KEY = secrets["GROQ_API_KEY"]

# Save the API key to environment variable
os.environ["GROQ_API_KEY"] = GROQ_API_KEY

INITIAL_RESPONSE = secrets["INITIAL_RESPONSE"]
INITIAL_MSG = secrets["INITIAL_MSG"]
CHAT_CONTEXT = secrets["CHAT_CONTEXT"]
```

In the `try block` we are getting the environment variables from the `.env` file to run it and test it locally.
But when we'll deploy it using streamlit we will not get any access of the `.env` file. So that time we will store our secrets using streamlit and to access those secrets we will use `st.secretes` that returns a python `dict`, same like `dotenv_values(".env")`. So after deployment the `except block` gets executed.

## Initializing the Chat Application

Let's set up the chat history and initialize the AI model:

- Copy your favourite AI-model's `Model ID` from [https://console.groq.com/docs/models](https://console.groq.com/docs/models):

![Groq supported models](./images/3.png)

I used `llama-3.1-8b-instant` for my project.

- Initialize your model:

```python
# Initialize the chat history if present as Streamlit session
if "chat_history" not in st.session_state:
    st.session_state.chat_history = [
        {"role": "assistant",
         "content": INITIAL_RESPONSE
         },
    ]

client = Groq()
```

We'll store the chat history in the st.session_state object, which allows us to persist data across session refreshes.

## Displaying the Chat Application

Let's create the chat interface using Streamlit:

```python
# Page title
st.title("Hey Buddy!")
st.caption("Let's go back in time...")

# Display chat history
for message in st.session_state.chat_history:
    with st.chat_message("role", avatar=''):
        st.markdown(message["content"])
```

We'll use the st.chat_message function to display each message in the chat history.

## User Input Field

Let's create a text input field for the user to enter their question:

```python
user_prompt = st.chat_input("Let's chat!")
```

When the user submits their prompt, we'll append it to the chat history and generate a response from the AI model.

## Generating a Response from the AI Model

Let's create a response from the AI model using the Groq library:

```python
def parse_groq_stream(stream):
    for chunk in stream:
        if chunk.choices:
            if chunk.choices[0].delta.content is not None:
                yield chunk.choices[0].delta.content

if user_prompt:
    with st.chat_message("user", avatar=""):
        st.markdown(user_prompt)
    st.session_state.chat_history.append(
        {"role": "user", "content": user_prompt})

    messages = [
        {"role": "system", "content": CHAT_CONTEXT
         },
        {"role": "assistant", "content": INITIAL_MSG},
        *st.session_state.chat_history
    ]

    stream = client.chat.completions.create(
        model="llama-3.1-8b-instant",
        messages=messages,
        stream=True  # for streaming the message
    )
    response = st.write_stream(parse_groq_stream(stream))
    st.session_state.chat_history.append(
        {"role": "assistant", "content": response})
```

We'll use the `client.chat.completions.create()` method to generate a steam and then parse it to a actual response from the AI model, and then append it to the chat history.

## Run it locally

Congratulations! You've built a personalised AI-powered chat application using Streamlit, Groq, and a llama-3.1-8b-instant model.

Here is the whole `main.py` file:

```python
import os
from dotenv import dotenv_values
import streamlit as st
from groq import Groq


def parse_groq_stream(stream):
    for chunk in stream:
        if chunk.choices:
            if chunk.choices[0].delta.content is not None:
                yield chunk.choices[0].delta.content


# streamlit page configuration
st.set_page_config(
    page_title="The Tech Buddy 🧑‍💻",
    page_icon="🤖",
    layout="centered",
)


try:
    secrets = dotenv_values(".env")  # for dev env
    GROQ_API_KEY = secrets["GROQ_API_KEY"]
except:
    secrets = st.secrets  # for streamlit deployment
    GROQ_API_KEY = secrets["GROQ_API_KEY"]

# save the api_key to environment variable
os.environ["GROQ_API_KEY"] = GROQ_API_KEY

INITIAL_RESPONSE = secrets["INITIAL_RESPONSE"]
INITIAL_MSG = secrets["INITIAL_MSG"]
CHAT_CONTEXT = secrets["CHAT_CONTEXT"]


client = Groq()

# initialize the chat history if present as streamlit session
if "chat_history" not in st.session_state:
    # print("message not in chat session")
    st.session_state.chat_history = [
        {"role": "assistant",
         "content": INITIAL_RESPONSE
         },
    ]

# page title
st.title("Welcome Hacker!")
st.caption("Helping You Level Up Your Coding Game.")
# the messages in chat_history will be stored as {"role":"user/assistant", "content":"msg}
# display chat history
for message in st.session_state.chat_history:
    # print("message in chat session")
    with st.chat_message("role", avatar='🤖'):
        st.markdown(message["content"])


# user input field
user_prompt = st.chat_input("Ask me")

if user_prompt:
    # st.chat_message("user").markdown
    with st.chat_message("user", avatar="🗨️"):
        st.markdown(user_prompt)
    st.session_state.chat_history.append(
        {"role": "user", "content": user_prompt})

    # get a response from the LLM
    messages = [
        {"role": "system", "content": CHAT_CONTEXT
         },
        {"role": "assistant", "content": INITIAL_MSG},
        *st.session_state.chat_history
    ]

    # Display assistant response in chat message container
    with st.chat_message("assistant", avatar='🤖'):
        stream = client.chat.completions.create(
            model="llama-3.1-8b-instant",
            messages=messages,
            stream=True  # for streaming the message
        )
        response = st.write_stream(parse_groq_stream(stream))
    st.session_state.chat_history.append(
        {"role": "assistant", "content": response})

```

To run it locally enter the following command in your terminal:

```bash
streamlit run main.py
```

## Deployment

We are now all set to deploy our app.
First upload the codebase in a [GitHub](https://github.com/) repository.
Then [click here](https://streamlit.io/) to sign in to your streamlit account and go to `My Apps` section:

- Click on `Create app` at the upper right corner.
  ![Create streamlit app](/images/4.png)

- Click on first option:
  ![streamlit create app](/images/5.png)

- Locate your github repository:
  ![streamlit create app - locate your github repo](/images/6.png)

- Locate the your `main.py` file:
  ![streamlit create app - locate your app file](/images/6.png)

- Create a custom url for your deployed app(optional):
  ![streamlit create app - create custom url](images/6.png)

- Click on `additional settings` and paste everything from your `.env` file (this is the `st.secrets`):
  ![streamlit create app - configure the secrets](/images/7.png)

- Click on deploy:

Congrats! you have successfully deployed your own personalised AI app for free.