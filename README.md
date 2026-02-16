# 🐾 Assistente Inteligente da Clínica Veterinária

Este projeto é um **assistente de IA para uma clínica veterinária**, desenvolvido como **Trabalho Final de Processamento de Linguagem Natural (PLN)**.

A ideia é que a clínica possa **fazer perguntas em linguagem natural** sobre seus próprios dados (atendimentos, animais, vacinas, clientes etc.) e o sistema responda com base nos **documentos internos**.

Ele usa a abordagem de **RAG – Retrieval-Augmented Generation**:

> 1. Busca primeiro nas bases vetoriais da clínica (Qdrant)  
> 2. Depois chama o modelo da OpenAI com esses trechos como contexto  
> 3. A resposta vem ancorada nos dados reais da instituição

---

## 💬 O que o assistente consegue responder?

Alguns exemplos de perguntas que o sistema entende e responde:

- **Quais tipos de atendimento existe?**  
- **Quais animais fazem aniversário em 11/10?**  
- **Quais vacinas estão ativas?**  
- **O animal Scooby tem vacinas programadas?**

Essas respostas são construídas a partir de coleções específicas da clínica:

- `atendimento_clinico` – tipos de atendimento, internações, vacinas e histórico clínico  
- `gestao_clinica` – inventário, formas de pagamento, agenda e organização da clínica  
- `informacoes_animais` – raças, patologias, vacinas, relatórios de vacinação  
- `marketing_clientes` – origem dos clientes, campanhas e informações de marketing  

---

## 🧠 Visão geral do projeto

### Objetivo

Criar um **assistente conversacional especializado** na rotina de uma clínica veterinária, capaz de:

- Centralizar informações importantes em um só lugar  
- Recuperar rapidamente dados de clientes, animais, atendimentos e vacinas  
- Apoiar a tomada de decisão da equipe com respostas consistentes e contextuais  

### Componentes principais

- **Aplicação Flask** 🐍  
  Interface web simples em formato de chat, onde o usuário envia perguntas.

- **n8n** ⚙️  
  Responsável por orquestrar os **agentes de IA**:  
  - decide quais coleções consultar  
  - chama Qdrant  
  - chama OpenAI  
  - combina tudo em uma resposta final.

- **Qdrant** 🧱  
  Banco de vetores que guarda os textos da clínica de forma indexada para busca semântica.

- **Postgres** 🗄️  
  Usado como **memória de conversa**, para o assistente lembrar do contexto recente.

- **Docker Compose** 🐳  
  Orquestra todos os serviços para subir tudo com um único comando.

Toda a estrutura de dados persistentes (Qdrant, Postgres etc.) está na pasta **`volumes/`**, conforme o requisito do projeto.

---

## 🧩 Arquitetura resumida

Fluxo simplificado de uma pergunta:

1. O usuário faz uma pergunta no chat (ex.: _“O animal Scooby tem vacinas programadas?”_).
2. A aplicação Flask envia a pergunta para um **Webhook do n8n**.
3. No n8n, um **Agente Proxy** analisa a pergunta e decide:
   - quais coleções do Qdrant usar (`informacoes_animais`, `atendimento_clinico`, etc.)
   - se precisa de memória de conversa (Postgres) ou não.
4. As ferramentas de QA do n8n consultam o Qdrant e retornam os **trechos mais relevantes**.
5. Esses trechos são enviados ao modelo da **OpenAI**, junto com a pergunta original.
6. O modelo gera uma resposta em linguagem natural, usando o contexto vindo das coleções.
7. A resposta é devolvida para a interface do chat e exibida ao usuário.

Assim, o assistente se comporta como um “atendente digital” especializado na rotina da clínica.

---

## 🛠️ Tecnologias utilizadas

- **Python + Flask** – backend e interface web  
- **OpenAI** – modelo de linguagem para geração de respostas  
- **Qdrant** – banco vetorial para RAG  
- **Postgres** – armazenamento da memória de chat  
- **n8n** – orquestração dos agentes e ferramentas  
- **Docker & Docker Compose** – empacotamento e orquestração dos serviços  

---

## 🚀 Como rodar o projeto

### 1. Pré-requisitos

- Docker  
- Docker Compose  

### 2. Configurar as variáveis de ambiente

Na pasta raiz do projeto (`pln`):

```bash
cp env.example .env
Depois, abra o arquivo .env e preencha:

OPENAI_API_KEY – chave da API da OpenAI

QDRANT_API_KEY e QDRANT_URL – credenciais/URL do Qdrant

demais variáveis usadas no ambiente (se definidas no env.example)

🔒 O arquivo .env não acompanha o projeto por segurança.
Cada ambiente deve preencher suas próprias chaves.

3. Subir os serviços
Ainda na pasta pln:

bash
Copiar código
docker compose up -d
Isso irá subir:

app Flask (chat)

n8n

Qdrant

Postgres

MinIO (se configurado)

4. Acessar a aplicação
Chat da clínica:
👉 http://localhost:5000

n8n (para inspecionar e editar os fluxos):
👉 http://localhost:5678

As coleções do Qdrant e os dados do banco já são carregados a partir da pasta volumes/, sem necessidade de reprocessar os arquivos.

🗂️ Estrutura de pastas (resumo)
text
Copiar código
src/           # lógica da aplicação (Flask, integrações)
templates/     # páginas HTML do chat
static/        # CSS, JS, imagens
uploads/       # uploads (se utilizados nos fluxos)
scripts/       # scripts auxiliares (ex.: ingestão/atualização de dados)
docs/          # documentação adicional
volumes/       # dados persistentes (Qdrant, Postgres, etc.)
docker-compose # arquivo de orquestração dos serviços
Dockerfile     # build da aplicação web
env.example    # modelo de arquivo .env
README         # este arquivo
