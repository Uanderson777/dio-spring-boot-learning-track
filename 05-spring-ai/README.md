# Projeto 05 — Spring AI com Groq (Budgeting)

Projeto educacional da DIO que integra Spring AI a uma API de controle financeiro. O código mantém a separação original entre domínio, casos de uso e infraestrutura.

## Requisitos

- JDK 25 instalado e configurado.
- MySQL 9.6 local, com o banco `budgeting` e a conta `budgeting_app` já configurados.
- Chave da Groq, configurada somente como variável de ambiente `GROQ_API_KEY`.
- Acesso à internet para baixar dependências Gradle e acessar a API da Groq.

O projeto conecta-se ao MySQL local em `localhost:3306` sem iniciar Docker. O Hibernate usa `ddl-auto=validate` e não cria nem atualiza tabelas.

> **Importante:** abra e execute este módulo a partir da pasta `05-spring-ai`. O repositório contém outros projetos independentes. Executar uma classe de outro módulo (por exemplo, `dio.catalog.Application`) não inicia esta aplicação.

## Abrir no IntelliJ IDEA

1. Abra a pasta `05-spring-ai` como projeto Gradle (não selecione a pasta de outro módulo).
2. Em **File > Project Structure > Project**, selecione o JDK 25.
3. Aguarde a sincronização do Gradle.
4. Para executar pela IDE, abra `src/main/java/dio/budgeting/BudgetingApplication.java`.
5. Clique em executar na classe `BudgetingApplication`.
6. Na configuração de execução, confirme:
   - **Main class:** `dio.budgeting.BudgetingApplication`
   - **Working directory:** a pasta absoluta do módulo `05-spring-ai`
   - **Environment variables:** `GROQ_API_KEY` com sua chave válida e `SPRING_DATASOURCE_PASSWORD` com a senha existente de `budgeting_app`.

O usuário padrão é `budgeting_app` e a URL padrão é `jdbc:mysql://localhost:3306/budgeting`. Não coloque chaves ou senhas no código, em `application.properties`, no README ou em arquivos versionados. Configure-as no ambiente local de execução da IDE.

## Executar pelo Windows (PowerShell)

Na raiz do repositório, entre primeiro no módulo:

```powershell
cd .\05-spring-ai
```

Confira o Java:

```powershell
java -version
```

O Gradle Wrapper do próprio módulo já está incluído. Compile:

```powershell
.\gradlew.bat clean compileJava
```

Configure `GROQ_API_KEY` e `SPRING_DATASOURCE_PASSWORD` nas variáveis de ambiente do Windows ou na configuração de execução do IntelliJ IDEA. Para solicitá-las sem eco e mantê-las somente na sessão atual do PowerShell:

```powershell
function Set-SessionSecret([string]$Name, [string]$Prompt) {
    $secure = Read-Host $Prompt -AsSecureString
    $pointer = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($secure)
    try {
        [Environment]::SetEnvironmentVariable(
            $Name,
            [Runtime.InteropServices.Marshal]::PtrToStringBSTR($pointer),
            [EnvironmentVariableTarget]::Process
        )
    } finally {
        [Runtime.InteropServices.Marshal]::ZeroFreeBSTR($pointer)
    }
}

$env:SPRING_DATASOURCE_USERNAME = "budgeting_app"
Set-SessionSecret "SPRING_DATASOURCE_PASSWORD" "Senha MySQL de budgeting_app"
Set-SessionSecret "GROQ_API_KEY" "Chave da Groq"
```

Compile, teste e inicie a aplicação:

```powershell
.\gradlew.bat compileJava
.\gradlew.bat test
.\gradlew.bat bootRun
```

Para encerrar, use `Ctrl+C`.

## Executar no Linux ou macOS

Dentro da pasta `05-spring-ai`, configure as credenciais sem gravá-las em arquivos:

```bash
read -s -p "Senha MySQL de budgeting_app: " SPRING_DATASOURCE_PASSWORD; echo
export SPRING_DATASOURCE_PASSWORD
export SPRING_DATASOURCE_USERNAME=budgeting_app
read -s -p "Chave da Groq: " GROQ_API_KEY; echo
export GROQ_API_KEY
./gradlew clean compileJava
./gradlew test
./gradlew bootRun
```

## Endpoints disponíveis

A aplicação usa a porta padrão `8080`.

| Método | Endpoint | Descrição |
|---|---|---|
| `POST` | `/transactions` | Registra uma transação por JSON |
| `GET` | `/transactions/{category}` | Lista transações por categoria |
| `POST` | `/transactions/ai` | Recebe áudio multipart no campo `file`, transcreve, processa com ferramentas e devolve áudio WAV |

Categorias definidas no domínio: `GROCERIES`, `PHARMA` e `AUTO`.

## Testes

Com o MySQL local ativo e `SPRING_DATASOURCE_PASSWORD` e `GROQ_API_KEY` configuradas, execute:

```powershell
.\gradlew.bat test
```

A compilação (`compileJava`) não precisa iniciar o banco nem chamar a Groq. Os testes de integração que chamam os serviços da Groq dependem de uma chave válida e de acesso à internet. Os testes de contexto também inicializam a aplicação e, portanto, dependem da configuração e dos serviços exigidos por ela.

## Configurações principais

- `build.gradle`: versões do Spring Boot, Spring AI, Java e dependências.
- `src/main/resources/application.properties`: configurações da aplicação, JPA e API compatível com OpenAI da Groq. A chave é lida de `GROQ_API_KEY`.
- `compose.yml`: mantido como material da aula; não é usado para executar a aplicação.
- `src/main/resources/prompts/system-message.st`: instrução de sistema do assistente financeiro.

O chat usa o modelo Groq `openai/gpt-oss-120b`; transcrição usa `whisper-large-v3-turbo`. A síntese usa `canopylabs/orpheus-v1-english`, atualmente limitada a inglês e WAV, com entrada de até 200 caracteres.

## Arquitetura preservada

- `domain`: entidades, identificadores, categorias e contrato do repositório.
- `application`: casos de uso para persistir e consultar transações.
- `infrastructure/http`: endpoints REST, transcrição de áudio, chamada do ChatClient e geração de áudio.
- `infrastructure/persistence`: adaptação JPA para persistência no MySQL.

O Spring AI utiliza os casos de uso como ferramentas, mantendo as regras de negócio fora do controlador e do domínio.

## Solução de problemas

- **A IDE inicia `dio.catalog.Application`:** a configuração selecionada pertence a outro módulo. Execute `dio.budgeting.BudgetingApplication`.
- **Erro de conexão com MySQL:** confira se o MySQL local está ativo em `localhost:3306`, se o banco `budgeting` existe e se `budgeting_app` tem acesso ao schema.
- **Erro de autenticação:** configure a senha existente de `budgeting_app` em `SPRING_DATASOURCE_PASSWORD`; o projeto não redefine senhas nem cria usuários.
- **Erro de chave da Groq:** confira se `GROQ_API_KEY` está definida no mesmo ambiente em que a aplicação é executada. Não cole a chave em arquivos versionados.
- **Áudio de resposta:** a Groq oferece atualmente síntese Orpheus em inglês e árabe, com saída WAV; respostas em português podem não ser pronunciadas corretamente.

## Referências

- [Spring AI — documentação oficial](https://docs.spring.io/spring-ai/reference/index.html)
- [Spring Boot — documentação oficial](https://docs.spring.io/spring-boot/index.html)
- [Groq — modelos disponíveis](https://console.groq.com/docs/models)
- [Groq — compatibilidade com OpenAI](https://console.groq.com/docs/openai)
