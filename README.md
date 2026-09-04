# 🎂 Pipeline Automatizado de Mensagens de Aniversário (n8n + LLM + WAHA + Google Sheets)

Este projeto consiste em um fluxo de automação desenvolvido no n8n para gerenciamento e envio automatizado de mensagens personalizadas de aniversário via WhatsApp, utilizando Inteligência Artificial.

---

## 🎨 Legendas de Cores (Canvas n8n)

Para manter a organização e facilitar a leitura do fluxo, os nós foram categorizados com as seguintes cores:

* **🟩 Verde (Definição de dados):** Entrada e leitura de dados externos (Formulários, Banco de Dados e Triggers).
* **🟥 Vermelho (Pré-condição / Verificação de sanidade):** Regras de negócio, validações e filtros de fluxo (`IF`).
* **🟦 Azul (Ação Executada):** Processamento ativo, chamadas de IA (`LLM`), envio de mensagens e gravação final.
* **🟧 Laranja (Nota / Comentário):** Orientações técnicas e requisitos de infraestrutura.

---

## 📝 1. Fluxo de Cadastro

Responsável pela entrada de novos contatos no sistema. Ele captura os dados enviados pelos usuários via formulário e registra ou atualiza as informações na base de dados.

### Componentes:
* **Form de Cadastro (Verde):** Gatilho (`n8n Form Trigger`) disponibilizado para coleta de dados do usuário (Nome, Telefone, Data de Nascimento e E-mail).
* **Cadastro de Usuário (Verde):** Nó do Google Sheets configurado em `appendOrUpdate`. Insere um novo contato ou atualiza o registro existente na planilha.

---

## ⚡ 2. Fluxo Principal (Disparo Automatizado)

Pipeline central de automação. Executa diariamente para identificar os aniversariantes do dia, gera uma mensagem única usando IA e realiza o envio pelo WhatsApp.

### Componentes:
* **Disparo diário (Verde):** Gatilho temporal (`Cron`) configurado para iniciar a execução uma vez ao dia.
* **Listar Usuários (Verde):** Consulta a planilha do Google Sheets e puxa a lista completa de contatos.
* **Valida Data de Aniversário (Vermelho):** Filtra apenas os contatos onde o dia e mês da data de nascimento coincidem com a data atual (`MM-DD`).
* **Valida Status do Envio (Vermelho):** Garante que o disparo só continue se o status for diferente de `'ENVIADO'`.
* **Base de Prompts & Gerador de Texto (Azul):** Conectado ao nó **Conector LLM**, envia os dados do aniversariante para a IA criar uma mensagem personalizada.
* **WhatsApp do Envio - WAHA (Azul):** Requisição `POST` que envia o texto gerado para a API do WAHA.
* **Atualizar Status para Enviado (Azul):** Atualiza a linha do contato na planilha para `'ENVIADO'`, finalizando o ciclo.

---

## 📌 Regras de Negócio e Infraestrutura (Notas)

* **Sanidade das Datas:** A validação compara apenas `MM-DD` atual para evitar falhas por conta do ano de nascimento.
* **Regra da IA:** O prompt é parametrizado para gerar mensagens curtas, humanizadas e sem excesso de emojis.
* **Requisito Técnico:** O envio de mensagens exige um container Docker ativo rodando a API do WAHA apontando para o endpoint `/api/sendText`.
