# Transfer Family

O projeto foi desenvolvido com o intuito de aplicar o AWS Transfer Family utilizando uma stack de autenticação de usuários com senhas via Lambda.

![transfer-family](https://github.com/BeatrizSantos-Costa/aws-soluctions/assets/64608296/002b943b-235e-425e-ad8a-d0c5049c13f1)

## Setup 

- [x] Acesso a AWS Console.

- [x] [AWS-CLI](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html) caso você opte por rodar o projeto pelo terminal.


## 🛠️ Aplicar a solução

*Para aplicar a solução é necessário estar logado na AWS*

1. Faça o clone do repositório
2. Acesse a console da AWS e navegue até o recurso S3 (Simple Storage Service)
3. Crie um bucket e após a criação realizar a subida do código Lambda que se encontra na pasta Lambda. (Vamos usar esse bucket na criação da Stack do CloudFormation)
4. Navegue até o recurso CloudFormation
3. Clique em `Create Stack` e escolha a opção `With new resources (standard)`
4. Em `Prerequisite - Prepare template` selecione a opção `Template is ready`
5. No campo `Template source` selecione `Upload Template File` e faça o upload do código que se encontra na pasta CloudFormation
6. Preencha as informações restantes e crie a stack.


### Criação do usuário no Secret Manager 

Vamos acessar o Secret Manager, onde vamos adicionar a secret em texto, segue abaixo exemplo de configuração:

1. Secrets Manager > Create Secret > Secret Type: Other type of secret > Adicione o seguinte texto e edite conforme preferir:

```json
{
 "Password": "MINHASENHA",
 "Role":"arn:aws:iam::ACCOUNT_ID:role/ROLE-DO-USER",
 "HomeDirectoryType":"LOGICAL",
 "HomeDirectoryDetails":"[{\"Entry\": \"/PASTA-DO-BUCKET\", \"Target\": \"/NOME-DO-BUCKET/PASTA-DO-BUCKET \"}]"
}

```

2. Em Configure Secret, o nome do segredo tem que ser o id do Transfer Family e o usuário, como o exemplo abaixo:


```
TRANSFER-FAMILY-ID/USERNAME
```

> O restante da configuração pode ser deixada como default


### Transferir aquivo utilizando client:

#### CyberDuck

1. Abra o cliente [CyberDuck](https://cyberduck.io/download/).
2. Escolha `Open Connection`.
3. Na caixa de diálogo `Open Connection`, escolha `SFTP (SSH File Transfer Protocol)`.
4. Para `Servidor`, insira o endpoint do seu servidor, ou seja, o endpoint do Transfer Family. **O endpoint do servidor está localizado na interface do Transfer Family**.
5. Em Número da porta, insira `22` para `SFTP`.
6. Em `Username`, insira o nome do usuário que você criou no **Secret Manager**.
7. Em `Password`, insira a senha definida no Secret Manager.
8. Escolha `Connect`.


#### SSH

Acesse o terminal da sua máquina e rode o comando abaixo:

```shell
sftp -i transfer-key USER_NAME@ENDPOINT_TRANSFER_FAMILY
```
