# Repositório de Configurações da Plataforma

Este repositório é a **fonte única de verdade** para todas as configurações da **Plataforma Multimodular SaaS**. Ele é o backend para o nosso `config-server` (baseado no Spring Cloud Config) e centraliza as propriedades de todos os microservices.

## 1\. Visão Geral e Propósito

O objetivo deste repositório é desacoplar as configurações do código-fonte das aplicações. Em vez de cada microserviço ter seu próprio arquivo `application.yml` com dados de conexão, portas e outras propriedades, todos eles buscam suas configurações neste local centralizado ao iniciar.

**Principais Vantagens:**

  * **Centralização:** Todas as configurações da plataforma em um único lugar.
  * **Versionamento:** Todas as alterações são controladas pelo Git. É possível auditar quem mudou o quê, quando, e reverter para versões anteriores facilmente.
  * **Segurança:** Informações sensíveis (senhas, chaves de API, etc.) não ficam expostas nos repositórios de código dos serviços.
  * **Gestão de Ambientes:** Permite gerenciar facilmente configurações para diferentes ambientes (desenvolvimento, homologação, produção) usando o sistema de *profiles* do Spring.

## 2\. Estrutura e Convenções de Nomenclatura

Para que o `config-server` encontre os arquivos corretos, seguimos uma convenção de nomenclatura estrita.

### 2.1. Nomes dos Arquivos

Os arquivos de configuração devem seguir um dos seguintes formatos:

  * **`application.yml`**: Define propriedades **globais**, que são aplicadas a **todos** os serviços que consomem este Config Server. É útil para configurações comuns, como endpoints do Actuator ou perfis de log.

  * **`{nome-da-aplicacao}.yml`**: Define propriedades específicas para um único serviço, que valem para **todos os ambientes**. O `{nome-da-aplicacao}` deve ser exatamente o mesmo valor definido na propriedade `spring.application.name` do microserviço.

      * **Exemplo:** `tenant-service.yml` contém as configurações padrão do `tenant-service`.

  * **`{nome-da-aplicacao}-{profile}.yml`**: Define ou sobrescreve propriedades para um serviço em um **ambiente específico**. O `{profile}` corresponde ao perfil Spring ativo (ex: `development`, `production`).

      * **Exemplo:** `tenant-service-production.yml` conteria a URL do banco de dados de produção, sobrescrevendo a de desenvolvimento.

### 2.2. Exemplo de Estrutura de Arquivos

```
plataforma-config/
├── 📄 application.yml               # Configurações globais para todos os serviços
|
├── 📄 tenant-service.yml             # Configurações padrão do Tenant Service
├── 📄 tenant-service-production.yml  # Sobrescreve configs do Tenant Service para o ambiente de produção
|
├── 📄 discovery-server.yml          # Configurações padrão do Discovery Server
|
└── ... (outros arquivos de configuração)
```

## 3\. Ordem de Prioridade das Propriedades

O Spring Cloud Config combina as propriedades de múltiplos arquivos de forma inteligente. A ordem de prioridade é a seguinte (o mais específico vence):

1.  **Propriedades específicas de profile e aplicação** (ex: `tenant-service-production.yml`)
2.  **Propriedades específicas da aplicação** (ex: `tenant-service.yml`)
3.  **Propriedades globais** (ex: `application.yml`)

**Exemplo Prático:**

  * Se `application.yml` define `logging.level.root: INFO`.
  * E `tenant-service.yml` define `logging.level.root: DEBUG`.
  * O `tenant-service` usará o nível `DEBUG`, pois a configuração específica da aplicação sobrescreve a global.

## 4\. Como Fazer Alterações

Qualquer alteração nas configurações da plataforma deve seguir este fluxo para garantir controle e segurança:

1.  Clone este repositório localmente.
2.  Crie uma nova branch para sua alteração (ex: `feature/ajuste-porta-gateway`).
3.  Edite ou crie o arquivo `.yml` necessário.
4.  Faça o commit da sua alteração com uma mensagem clara.
5.  Envie a branch para o repositório remoto (`git push`).
6.  Abra um **Pull Request** para que a alteração seja revisada por outro membro da equipe.

Uma vez que o Pull Request for aprovado e o merge para a branch principal (`main`) for feito, o `config-server` passará a servir as novas propriedades para os serviços na próxima vez que eles forem reiniciados (ou quando o endpoint `/actuator/refresh` for acionado).

## 5\. Verificando as Configurações Servidas

Para inspecionar em tempo real quais propriedades o `config-server` está servindo para uma determinada aplicação, você pode usar a API dele.

  * **Formato da URL:** `http://localhost:8888/{nome-da-aplicacao}/{profile}`

**Exemplo:**
Para ver as configurações que o `tenant-service` receberá no perfil `default`, acesse no navegador:
[http://localhost:8888/tenant-service/default](https://www.google.com/search?q=http://localhost:8888/tenant-service/default)

A resposta será um JSON detalhando as fontes de propriedades e os valores que estão sendo aplicados.