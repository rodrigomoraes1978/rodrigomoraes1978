# 🏛️ Enterprise Multi-Cloud Architecture: Resilient GenAI Platform

Este repositório contém a especificação arquitetural de uma plataforma de Inteligência Artificial Generativa e mensageria em larga escala, projetada sob os pilares de **Privacidade por Design (LGPD)**, **FinOps (Otimização de Custos)** e **Alta Disponibilidade (High-Throughput)**.

A arquitetura foi desenhada para processar webhooks de mensageria (como a API Oficial do WhatsApp Meta) sem perda de pacotes, tratando picos massivos de concorrência através de um fluxo assíncrono e isolado de rede.

---

## 🏗️ Visão Geral da Arquitetura (Topology)

```mermaid
graph TD
    User([Usuário / Webhook Meta]) -->|Tráfego HTTPS| Nginx[Nginx Proxy Reverso & Rate Limiter]
    
    subgraph DMZ / Borda Pública
        Nginx
    end

    subgraph VPC Isolada / Rede Local Privada (GCP & OCI)
        Nginx -->|Roteamento Interno: IP & Portas| Kafka[(Apache Kafka Message Broker)]
        Kafka -->|Backpressure Handling| NodeMS[Microsserviços Ingestão - Node.js]
        NodeMS -->|Desacoplamento| PyMS[Microsserviços IA & Guardrails - Python]
        
        subgraph Pipeline de Orquestração de LLMs
            PyMS -->|Filtro 1: Input Guardrails| Input[Validação & Anti-Prompt Injection]
            Input -->|Filtro 2: Roteamento de Custo| Router{Model Routing Engine}
            Router -->|Demanda Simples / Custo Zero| Llama[Llama Local Instance]
            Router -->|Demanda Complexa / Escalonada| GPT[ChatGPT Enterprise API]
            Llama --> Output[Filtro 3: Output Guardrails & PII Masking]
            GPT --> Output
        end
    end

    Output -->|Payload Criptografado| ReactApp[Frontend Client - React & TS]
    ReactApp -->|Decodificação em Memória / F12 Blindado| User
```

---

## 🛠️ Pilares Técnicos & Engenharia de Infraestrutura

### 1. Segurança de Perímetro e Isolamento de Rede (Zero Trust)
* **Proxy Reverso (Nginx)**: O DNS público aponta exclusivamente para uma camada de borda gerenciada pelo Nginx. Ele atua como *Reverse Proxy* e aplica *Rate Limiting* para controlar o bombardeio de requisições e mitigar ataques.
* **Subnets Privadas Corporativas**: Todos os microsserviços (Node.js, Python), corretores de mensageria (Kafka) e instâncias de IA operam em servidores não-públicos com acesso estritamente local. O Nginx direciona o tráfego internamente via IPs e portas privadas, bloqueando qualquer varredura ou exposição externa direta.

### 2. Resiliência a Altas Cargas e Tratamento de Concorrência
* **Event-Driven Architecture (Apache Kafka)**: Para suportar picos de milhares de mensagens simultâneas, o sistema utiliza o Kafka como *Message Broker*. O payload dos webhooks é enfileirado instantaneamente de forma assíncrona, eliminando timeouts e garantindo perda zero de pacotes.
* **Controle de Pressão (Backpressure Handling)**: Os microsserviços consomem os tópicos do Kafka de acordo com a capacidade computacional disponível, protegendo as APIs de IA de estouros de limites de taxa (*Rate Limits*) dos provedores.

### 3. Criptografia na Camada de Aplicação (Client-Side Protection)
* **End-to-End App Encryption**: Visando a conformidade estrita com a LGPD para dados sensíveis, o fluxo de dados entre os microsserviços e o frontend em **React** utiliza criptografia direta na camada de aplicação.
* **Obfuscação de Payload**: Os pacotes de dados trafegam pela rede de forma cifrada e totalmente ilegível. Mesmo que um usuário inspecione a aba *Network* do navegador (F12), visualizará apenas strings protegidas. A decodificação ocorre estritamente em memória no cliente no momento da renderização.

### 4. Orquestração Híbrida de Modelos e Inteligência Artificial (FinOps)
* **Model Routing (Roteamento por Custo)**: O ecossistema economiza milhares de dólares em infraestrutura ao não direcionar todas as requisições para LLMs proprietárias pagas. O sistema analisa a intenção do usuário: regras determinísticas e tarefas simples são processadas localmente por modelos **Llama** (Custo Zero), escalando para o **ChatGPT** apenas em demandas cognitivas complexas.
* **Filtros em Cascata (Guardrails)**: A resposta final passa por um pipeline rigoroso de validação em três etapas: validação do input do usuário, checagem de alucinação baseada no contexto e mascaramento automático de informações sensíveis (PII Masking) antes do envio.

---

## 📈 Tecnologias Consolidadas no Case
* **Frontend**: React, TypeScript, TailwindCSS (Memória isolada para decodificação).
* **Backend**: Node.js (Alta performance em I/O assíncrono), Python (Ecossistema de IA/LLMs), FastAPI.
* **Infraestrutura & Nuvem**: Multicloud Distribuída (GCP - Google Cloud e OCI - Oracle Cloud).
* **Mensageria & Proxy**: Apache Kafka, Nginx Core.
* **Modelos de Linguagem**: OpenAI GPT Enterprise, Meta Llama (Deploy local).
