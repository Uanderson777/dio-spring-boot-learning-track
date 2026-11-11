# Projeto 05 — Spring AI (Budgeting)

Projeto educacional da DIO que integra Spring AI a uma API de controle financeiro. O código mantém a separação original entre domínio, casos de uso e infraestrutura.

## Requisitos

- JDK 25 instalado e configurado.
- Docker Desktop instalado e em execução (o projeto usa MySQL pelo Docker Compose).
- Chave de API da OpenAI, configurada somente como variável de ambiente `OPENAI_API_KEY`.
- Acesso à internet para baixar dependências Gradle e acessar a API da OpenAI.

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
   - **Environment variables:** `OPENAI_API_KEY` com sua chave válida.

Não coloque a chave no código, em `application.properties`, no README ou em arquivos versionados. Configure-a nas variáveis de ambiente do sistema ou na configuração local de execução da IDE.

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

Configure `OPENAI_API_KEY` nas variáveis de ambiente do Windows ou na configuração de execução do IntelliJ IDEA. Para uma sessão temporária do PowerShell, defina a variável antes de iniciar a aplicação:

```powershell
$env:OPENAI_API_KEY = "SUA_CHAVE_OPENAI"
```

Substitua o texto de exemplo localmente. Não compartilhe capturas de tela que mostrem a chave e nunca faça commit desse valor.

Inicie o Docker Desktop e execute:

```powershell
.\gradlew.bat bootRun
```

O Spring Boot Docker Compose procura o arquivo `compose.yml` no diretório de trabalho da aplicação. Por isso, execute o comando estando dentro de `05-spring-ai`, onde esse arquivo está localizado. O Compose inicia o MySQL na porta `3307` do computador (porta `3306` dentro do contêiner).

Para encerrar, use `Ctrl+C`. O volume `transaction_data` mantém os dados do banco entre reinicializações.

## Executar no Linux ou macOS

Dentro da pasta `05-spring-ai`, configure `OPENAI_API_KEY` no ambiente e execute:

```bash
export OPENAI_API_KEY="SUA_CHAVE_OPENAI"
./gradlew clean compileJava
./gradlew bootRun
```

## Endpoints disponíveis

A aplicação usa a porta padrão `8080`.

| Método | Endpoint | Descrição |
|---|---|---|
| `POST` | `/transactions` | Registra uma transação por JSON |
| `GET` | `/transactions/{category}` | Lista transações por categoria |
| `POST` | `/transactions/ai` | Recebe áudio multipart no campo `file`, transcreve, processa com ferramentas e devolve áudio MP3 |

Categorias definidas no domínio: `GROCERIES`, `PHARMA` e `AUTO`.

## Testes

Com Docker Desktop ativo e a variável `OPENAI_API_KEY` configurada, execute:

```powershell
.\gradlew.bat test
```

A compilação (`compileJava`) não precisa iniciar o banco nem chamar a OpenAI. Os testes de integração que chamam os serviços da OpenAI dependem de uma chave válida e de acesso à internet. Os testes de contexto também inicializam a aplicação e, portanto, dependem da configuração e dos serviços exigidos por ela.

## Configurações principais

- `build.gradle`: versões do Spring Boot, Spring AI, Java e dependências.
- `src/main/resources/application.properties`: configurações da aplicação, JPA e modelos OpenAI. A chave é lida de `OPENAI_API_KEY`.
- `compose.yml`: serviço MySQL e volume persistente.
- `src/main/resources/prompts/system-message.st`: instrução de sistema do assistente financeiro.

## Arquitetura preservada

- `domain`: entidades, identificadores, categorias e contrato do repositório.
- `application`: casos de uso para persistir e consultar transações.
- `infrastructure/http`: endpoints REST, transcrição de áudio, chamada do ChatClient e geração de áudio.
- `infrastructure/persistence`: adaptação JPA para persistência no MySQL.

O Spring AI utiliza os casos de uso como ferramentas, mantendo as regras de negócio fora do controlador e do domínio.

## Solução de problemas

- **Aparece `No Docker Compose file found`:** confirme que o diretório de trabalho é `05-spring-ai` e que o arquivo `compose.yml` está nessa pasta.
- **A IDE inicia `dio.catalog.Application`:** a configuração selecionada pertence a outro módulo. Execute `dio.budgeting.BudgetingApplication`.
- **Erro de conexão com MySQL:** inicie o Docker Desktop e confira se a porta `3307` está livre.
- **Erro de chave da OpenAI:** confira se `OPENAI_API_KEY` está definida no mesmo ambiente em que a aplicação é executada. Não cole a chave em arquivos versionados.

## Referências

- [Spring AI — documentação oficial](https://docs.spring.io/spring-ai/reference/index.html)
- [Spring Boot — documentação oficial](https://docs.spring.io/spring-boot/index.html)
- [Docker Compose — documentação oficial](https://docs.docker.com/compose/)
