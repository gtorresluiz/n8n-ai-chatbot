# 🤖 Agente de Suporte Operacional com IA (n8n + Google Gemini)

Este projeto demonstra a criação de um **Agente de Suporte Operacional** totalmente automatizado, capaz de responder a dúvidas de clientes em tempo real, utilizando uma base de conhecimento interna simulada do IFood e inteligência artificial (Google Gemini).

O fluxo de trabalho (workflow) é construído na plataforma de automação **n8n**, garantindo que as respostas sejam rápidas, precisas e **validadas pelas instruções internas do agente**.

---

## ✨ Destaques do Projeto

* **POC de Agente Interno:** Desenvolvimento de uma Prova de Conceito (POC) de Agente de IA para decisões operacionais (reembolso/cancelamento), utilizando uma **base de conhecimento simulada** de políticas do iFood.
* **Processamento de Dados (RAG):** Demonstra a unificação de diferentes fontes de dados (JSON de requisição e Contexto Agrupado em CSV/Texto) utilizando o nó `Merge` para alimentar o modelo (uma abordagem de **Retrieval-Augmented Generation - RAG**).
* **Controle de Qualidade Integrado:** A lógica de *guard-rail* e o *fallback* para baixa confiança foram **diretamente instruídos no prompt do Agente LLM**. Isso simplifica o fluxo e aumenta a eficiência, pois o agente é o único responsável por decidir se a resposta é viável ou se deve usar a mensagem de fallback ("Não tenho política suficiente...").
* **Cenários Críticos Testados:** O fluxo foi testado com cenários de alto risco operacional, como pedidos já despachados para entrega, falhas do restaurante e cobranças indevidas após o cancelamento.
* **Arquitetura:** Criação de um fluxo de trabalho assíncrono e resiliente para tarefas de suporte.

---

## ⚙️ Arquitetura do Workflow

O fluxo é dividido em três etapas principais. A principal mudança é que a validação agora ocorre dentro do Agente (Etapa 2), e não em um nó `If` externo.

### 1. Preparação e Agrupamento de Dados

| Nó | Função |
| :--- | :--- |
| **Webhook** | Recebe a requisição inicial do usuário (a pergunta). |
| **Transformação de Dados** | Processa e agrupa a base de conhecimento (simulada) em uma única string (`meu_contexto_agrupado`). |
| **Set (Extrair Pergunta)** | Filtra o JSON do Webhook para extrair apenas a chave `pergunta_do_cliente`. |
| **Merge (Combine)** | Combina o objeto `pergunta_do_cliente` e o `meu_contexto_agrupado` em um **único objeto JSON**, pronto para ser injetado no LLM. |

### 2. Processamento da Inteligência Artificial e Decisão

| Nó | Função |
| :--- | :--- |
| **Google Gemini Chat Model** | O Agente de IA recebe o JSON combinado e atua como um Agente Operacional. **Ele contém toda a lógica de decisão:** *Responde APENAS com a Base de Conhecimento* e *aplica o fallback automático* se a resposta não for clara. |
| **Instrução Chave (Prompt)** | O prompt inclui a regra: "Se não encontrar resposta clara, responda: 'Não tenho política suficiente para responder isso. Encaminhe para suporte.'". Isso garante consistência operacional. |

### 3. Saída Controlada

| Nó | Função |
| :--- | :--- |
| **Set (JSON Parser)** | Garante que a resposta de texto do LLM (que vem formatada como JSON, incluindo a chave `resposta` e `confianca`) seja convertida em um **objeto JSON** utilizável. |
| **Set (Resposta Limpa)** | Extrai a mensagem final (`resposta` do JSON) para ser enviada ao usuário. |
| **Ação Final** | Envio da resposta ao canal de comunicação do cliente. |

---

## 🛠️ Como Executar Este Projeto

Para importar e executar este workflow em sua própria instância do n8n:

### Pré-requisitos

1.  Uma instância do **n8n** (self-hosted ou Cloud).
2.  Uma **API Key** válida do **Google Gemini** (pode ser obtida gratuitamente no Google AI Studio).

### Passos de Execução

1.  **Clone o Repositório:** Baixe ou clone este repositório.
2.  **Importe o Workflow:**
    * Abra sua interface do n8n.
    * No painel principal, clique em **"New" > "Import from JSON"**.
    * Selecione o arquivo `[nome_do_seu_projeto].json` que você exportou.
3.  **Configure as Credenciais:**
    * Clique no nó **Google Gemini Chat Model**.
    * Clique em **"Credentials"** e adicione sua **API Key do Gemini**.
4.  **Teste:** Execute o workflow ou envie uma requisição POST para o endereço do Webhook configurado.
 4.1.
   *  Utiliziando **POSTMAN**
   *  Método POST
     ```
      http://localhost:5678/webhook-test/ia-operacional
     ```
  * body
    ``` json
    {
      "pergunta": "O cliente pode pedir reembolso após o pedido sair para entrega?"
    }
    ```

 ---

---

## 🤝 Conecte-se

Fique à vontade para entrar em contato para discutir este projeto ou outras soluções de automação e IA!
