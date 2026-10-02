# 💰 RAG para Leitura de Documentos Financeiros

Um projeto de **RAG (Retrieval-Augmented Generation)** focado na leitura e interpretação de demonstrativos financeiros em PDF, combinando busca semântica, banco vetorial e IA generativa para responder perguntas sobre documentos contábeis e financeiros.

## 📋 Descrição do Projeto

Este repositório implementa um assistente inteligente para análise de documentos financeiros que:

- **Carrega PDFs** de demonstrativos financeiros e extrai texto
- **Divide documentos em chunks** para melhor recuperação e contexto
- **Cria embeddings** com modelos Hugging Face
- **Armazena vetores em ChromaDB** para busca semântica
- **Recupera trechos relevantes** para responder perguntas
- **Gera respostas** com LLM via Groq, usando apenas o contexto fornecido

## 🎯 Caso de Uso

Ideal para:
- 📊 **Analistas Financeiros**: consultar demonstrativos e extratos sem navegar manualmente em PDFs longos
- 🧠 **Consultores**: descobrir rapidamente informações específicas em documentos
- 📁 **Times de Contabilidade**: responder dúvidas estruturadas sobre receitas, despesas e fluxo de caixa
- 🧪 **Estudos de IA para documentos**: demonstrar RAG em arquivos financeiros reais
- 🎓 **Aprendizado de RAG**: aplicação prática com LangChain, embeddings e ChromaDB

## 🏗️ Arquitetura

```
┌──────────────────────────────────────────────────────────────┐
│                     Documento PDF                           │
│          Demonstrativo Financeiro / Balanço / DRE           │
└────────────────┬─────────────────────────────────────────────┘
                 │
         ┌───────▼────────────┐
         │  PyPDFLoader       │
         │  Extrai texto      │
         │  das páginas       │
         └───────┬────────────┘
                 │
         ┌───────▼────────────┐
         │  RecursiveTextSplitter │
         │  Divide em chunks  │
         │  (800 chars, 200 overlap) │
         └───────┬────────────┘
                 │
         ┌───────▼────────────┐
         │  Embedding Model   │
         │  sentence-transformers │
         └───────┬────────────┘
                 │
         ┌───────▼────────────┐
         │  ChromaDB          │
         │  Banco Vetorial    │
         │  persist_directory │
         └───────┬────────────┘
                 │
         ┌───────▼────────────┐
         │  Retriever         │
         │  Busca 3 chunks    │
         │  mais relevantes   │
         └───────┬────────────┘
                 │
         ┌───────▼────────────┐
         │  Prompt + Contexto │
         │  (RAG)             │
         └───────┬────────────┘
                 │
         ┌───────▼────────────┐
         │  Groq / LLM        │
         │  Resposta em PT-BR │
         └────────────────────┘
```

## 📁 Estrutura do Projeto

```
dsa_RAG_leitura_doc_financeiro/
├── dsa_app.py                    # Aplicação Streamlit principal
├── comparar_embeddings.py        # Script para comparar modelos de embedding
├── requirements.txt              # Dependências do projeto
├── .env.example                  # Exemplo de variáveis de ambiente (se existir)
├── chroma_db_persist/            # Banco vetorial persistido localmente
├── demonstrativo_financeiro.pdf  # Documento usado para testes (se presente)
└── README.md                     # Este arquivo
```

## 🚀 Como Executar

### 1️⃣ Pré-requisitos

- Python 3.10+
- pip
- Chave da API Groq
- Arquivo PDF financeiro para upload
- Ambiente virtual recomendado

### 2️⃣ Instalação

```bash
# Clone o repositório
git clone https://github.com/alinemiranda036/dsa_RAG_leitura_doc_financeiro.git
cd dsa_RAG_leitura_doc_financeiro

# Crie um ambiente virtual
python -m venv venv
source venv/bin/activate   # Linux/macOS
# ou
venv\Scripts\activate      # Windows

# Instale as dependências
pip install -r requirements.txt
```

### 3️⃣ Configurar a API da Groq

Crie um arquivo `.env` na raiz do projeto:

```bash
GROQ_API_KEY=sua_chave_aqui
```

### 4️⃣ Executar a aplicação

```bash
streamlit run dsa_app.py
```

A interface será aberta em `http://localhost:8501`.

### 5️⃣ Usar a aplicação

1. Faça upload de um PDF financeiro
2. Aguarde o processamento do documento
3. Faça uma pergunta em português sobre o arquivo
4. O sistema buscará chunks relevantes e gerará resposta

## 🔧 Componentes Principais

### 1. `dsa_app.py` - Aplicação principal

A aplicação Streamlit faz o pipeline completo de RAG:

- carrega o PDF enviado pelo usuário
- extrai texto com `PyPDFLoader`
- divide em chunks com `RecursiveCharacterTextSplitter`
- gera embeddings com `HuggingFaceEmbeddings`
- salva e recupera os dados no ChromaDB
- usa `ChatGroq` para responder perguntas com contexto

**Fluxo principal:**

```python
# 1. PDF -> documentos
loader = PyPDFLoader(tmp_file_path)
docs = loader.load()

# 2. Texto -> chunks
text_splitter = RecursiveCharacterTextSplitter(chunk_size=800, chunk_overlap=200)
splits = text_splitter.split_documents(docs)

# 3. Chunks -> embeddings
vectorstore = Chroma.from_documents(
    documents=splits,
    embedding=embedding_model,
    persist_directory=CHROMA_PERSIST_DIR,
    collection_name=CHROMA_COLLECTION_NAME,
)

# 4. Busca relevante
retriever = vectorstore.as_retriever(search_kwargs={"k": 3})

# 5. Geração da resposta
answer = dsa_rag_chain.invoke(question)
```

### 2. `comparar_embeddings.py` - Benchmark de modelos

Este script compara diferentes modelos de embeddings em cima do mesmo PDF e das mesmas perguntas.

**Objetivos:**
- testar qualidade de recuperação
- identificar qual modelo retorna melhor contexto
- comparar tempo de carregamento, indexação e busca

**Modelos comparados:**
- `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2`
- `intfloat/multilingual-e5-base`
- `BAAI/bge-m3`

### 3. `requirements.txt` - Dependências

Inclui bibliotecas essenciais para:

- processamento de PDFs
- embeddings
- banco vetorial
- framework RAG
- LLM via Groq
- interface web

## 🧠 Como o RAG funciona neste projeto

O fluxo é simples e prático:

1. O usuário envia um PDF financeiro
2. O texto é extraído e dividido em trechos menores
3. Cada trecho vira um vetor por embeddings
4. O banco vetorial indexa os trechos
5. A pergunta do usuário é transformada em embedding
6. O retriever busca os trechos mais relevantes
7. O LLM recebe esses trechos como contexto e responde

**Prompt usado:**

```python
RAG_PROMPT_TEMPLATE = """
Você é um assistente de IA especializado em análise financeira.
Sua tarefa é responder perguntas sobre demonstrativos financeiros usando APENAS o contexto fornecido.
Seja direto, preciso e baseie-se exclusivamente nos dados dos trechos.
Se a informação não estiver no contexto, diga "A informação não foi encontrada no documento."

Contexto:
{context}

Pergunta:
{question}

Resposta (em Português):
"""
```

Isso reduz significativamente alucinações e mantém a resposta aderente ao documento.

## 📚 Exemplos de Perguntas

Você pode perguntar coisas como:

- "Qual foi o critério de reconhecimento de receita?"
- "Qual o fluxo de caixa operacional?"
- "Qual o lucro líquido da empresa?"
- "Qual foi o valor do patrimônio líquido?"
- "Quanto foi investido em equipamentos?"

## ⚙️ Configuração Avançada

### Ajustar número de chunks recuperados

No `dsa_app.py`, a busca usa:

```python
return vectorstore.as_retriever(search_kwargs={"k": 3})
```

Você pode aumentar ou diminuir para:

```python
{"k": 5}
```

### Ajustar o tamanho dos chunks

```python
RecursiveCharacterTextSplitter(chunk_size=800, chunk_overlap=200)
```

Parâmetros úteis:
- `chunk_size`: tamanho do trecho
- `chunk_overlap`: sobreposição entre chunks

### Trocar o modelo de embedding

```python
EMBEDDING_MODEL = "sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2"
```

Você pode trocar por outros modelos de qualidade mais alta, como:
- `intfloat/multilingual-e5-base`
- `BAAI/bge-m3`

## 📊 Comparação de Modelos de Embedding

Este projeto inclui um script para comparar diferentes embeddings em termos de:

| Critério | Observação |
|---------|-----------|
| **Taxa de acerto** | Quão bem o modelo encontra respostas esperadas |
| **Tempo de carregamento** | Tempo para iniciar o modelo |
| **Tempo de indexação** | Tempo para criar os vetores |
| **Tempo de busca** | Velocidade de recuperação |

**Vantagem**: você consegue selecionar o melhor trade-off entre velocidade, custo e qualidade.

## 🛡️ Segurança e Privacidade

- ✅ O processamento do PDF acontece localmente
- ✅ O conteúdo fica persistido em banco vetorial local
- ✅ O LLM é acessado via Groq API
- ⚠️ Sempre verifique dados sensíveis antes de enviar para modelos externos
- ⚠️ O projeto é voltado para uso educacional e prototipagem

## 📈 Melhorias Futuras

- [ ] Suporte a múltiplos PDFs ao mesmo tempo
- [ ] Reconhecimento de tabelas estruturadas
- [ ] Extração de dados numéricos em formato tabular
- [ ] Comparação entre diferentes documentos e períodos
- [ ] Dashboard para análise financeira
- [ ] Integração com SQL/Delta Lake
- [ ] Cache inteligente de embeddings
- [ ] API REST para uso por outros sistemas
- [ ] Execução em GPU para maior performance

## 🤝 Contribuindo

Contribuições são bem-vindas! Você pode:
1. Melhorar o pipeline de leitura de PDFs
2. Adicionar novos modelos de embedding
3. Criar testes de qualidade de recuperação
4. Melhorar a interface UX/UI
5. Abrir issues e pull requests

## 📝 Licença

Este projeto foi desenvolvido para fins educativos e de demonstração.

## 📞 Suporte

Para dúvidas ou problemas:
- Abra uma issue no GitHub
- Consulte a documentação do LangChain: https://python.langchain.com/
- Consulte o ChromaDB: https://www.trychroma.com/
- Consulte a documentação da Groq: https://console.groq.com/docs
- Consulte os modelos Hugging Face: https://huggingface.co/

## 🎓 Recursos e Referências

- [LangChain Documentation](https://python.langchain.com/docs/)
- [Hugging Face Embeddings](https://huggingface.co/)
- [ChromaDB Docs](https://docs.trychroma.com/)
- [Groq API Docs](https://console.groq.com/docs)
- [PyPDF Documentation](https://pypdf.readthedocs.io/)
- [RAG Best Practices](https://python.langchain.com/docs/tutorials/retrievers/)

---

**Desenvolvido com ❤️ para a comunidade de IA aplicada a documentos financeiros**
