
import streamlit as st
from google import genai
from google.genai import types

st.set_page_config(
    page_title="Mentor de Carreiras IA",
    page_icon="🧭",
    layout="centered"
)

st.title("🧭 Mentor de Carreiras com IA")
st.subheader("Descubra seu caminho profissional!")

st.write("""
Sou seu mentor virtual de carreiras.
Vou ajudar você a identificar habilidades,
interesses e oportunidades profissionais.
""")

INSTRUCOES = """
Você é um mentor de carreiras profissional,
acolhedor e especializado em orientação profissional.

Sua missão é ajudar pessoas a:
- Descobrir seus talentos e habilidades.
- Identificar áreas profissionais compatíveis.
- Encontrar possibilidades de carreira.
- Desenvolver competências.
- Criar planos de desenvolvimento profissional.

Faça uma pergunta por vez.
Seja claro, motivador e objetivo.
Não prometa emprego ou salários.
Não invente cursos ou oportunidades.
Responda sempre em português brasileiro.
"""

api_key = st.secrets["GEMINI_API_KEY"]

client = genai.Client(api_key=api_key)

if "chat" not in st.session_state:
    st.session_state.chat = client.chats.create(
        model="gemini-2.5-flash",
        config=types.GenerateContentConfig(
            system_instruction=INSTRUCOES,
            temperature=0.7
        )
    )

if "mensagens" not in st.session_state:
    st.session_state.mensagens = []

for mensagem in st.session_state.mensagens:
    with st.chat_message(mensagem["role"]):
        st.markdown(mensagem["content"])

pergunta = st.chat_input("Digite sua mensagem...")

if pergunta:
    st.session_state.mensagens.append({
        "role": "user",
        "content": pergunta
    })

    with st.chat_message("user"):
        st.markdown(pergunta)

    with st.chat_message("assistant"):
        with st.spinner("Analisando seu perfil..."):
            try:
                resposta = st.session_state.chat.send_message(pergunta)
                texto = resposta.text
            except Exception as erro:
                texto = f"Erro ao consultar a IA: {erro}"

            st.markdown(texto)

    st.session_state.mensagens.append({
        "role": "assistant",
        "content": texto
    })

if st.button("🔄 Nova conversa"):
    del st.session_state.chat
    del st.session_state.mensagens
    st.rerun()
