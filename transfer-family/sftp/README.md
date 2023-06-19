
# Transfer Family - Criação de SFTP baseado em nome de usuário e senha utilizando o Lambda

Vamos aprender como realizar a autenticação de usuários com o Tranfer Family, pois de forma "padrão" ele utiliza o login via chave pública do SSH. 

## Criação Lambda

O Lambda vai ser o resposnsável por realizar a parte de autenticação, onde o mesmo vai verificar se a autenticação atende todos os requisitos para a atividade.

**Configurando o Lambda:**
 * Function Name: ``lambda-auth``
 * Runtime: ``Python 3.10``
 * Architecture: ``x86_64``
 * Execution Role: ``Create a new role with basic Lambda permissions``

> Faça o upload do código que se encontra no repositório.

##  Criação do Transfer Family

O AWS Transfer Family é um serviço de transferência seguro que permite transferir arquivos para dentro e fora dos serviços de armazenamento da AWS. Vamos utilizado para pegar dados localmente e levarmos para o S3. 

**Configurando o Transfer Family:**
 * Chose protocols: ``SFTP`` > ``Next``
 * Choose an Identity Provider: ``Custom Identity Provider`` > AWS Lambda Function: ``lambda-auth`` > ``Next`` 
 * Choose an endpoint: ``Publicly accessible`` > ``Next``
 * Choose a Domain: ``Amazon S3`` > ``Next``
 * Configure additional details: ``As opções Default já são suficientes para essa POC`` > ``Next`` > ``Create``

 ## Criação do S3

Nosso serviço de escolhido no Transfer Family é o S3 Bucket. Você pode criar o balde com sua preferência. 

> Minhas sugestões: bloqueie todo o acesso público e ative a criptografia e o controle de versão.

**Configurando o bucket:**
 * Nome do Bucket: ``poc-transfer-family-sftp``
 * ACL: bloqueando ``acesso público``
 * Criptografia: ``Ativo``
 * Controle de Versão: ``Ativo``

## Ajustando Permissões na Lambda

Vamos precisar ajustar as permissões no Lambda, pois não adicionamos as permissões corretas para a execução ser concluída com sucesso. 

1. Navegue até: 
    Lambda > Configuration > Permissions, scroll to "Resource-based policy" and click on "Add permissions". 
    
    ```
    AWS Serivce
    Service: Other
    Statement ID: lambda-auth-invocation-s-d3cf65819e2c47a6b
    Principal: transfer.amazonaws.com
    Source ARN: arn:aws:transfer:{REGION}:{ACCOUNT_ID}:server/{SERVER_ID_TF}
    Action: lambda:InvokeFunction
    ```
2. Vamos adicionar permissão na Lambda para ela conseguir acessar o Secret Manager, o código fica no repositório. 

    2.1 Acessar a role da Lambda > Clicar no "+" > Copiar a ARN do CloudWatch Logs

    2.2 Polices > Create Policy > Clique em "Json" > Adicione a policy que se encontra no repositório com o nome "policy-lambda-auth.json"

    2.3 Remover a policy antiga e adicionar a nova

## IAM Policy e Role Usuário

Precisamos criar uma Role e policy nas quais os usuários vão assumir para se conectar ao Transfer Family. 

1. Criação da Policy:

    1.1 IAM > Polices > Create Policy > Clique em "Json" 
    1.2 Adicione a policy que se encontra no repositório com o nome "policy-tf-user-acess.json".
    1.3 Policy Name: policy-tf-user-acess


2. Criação da Role:

    2.1 IAM > Roles > Create Role
    ```
        Selecione: AWS Service
        Use Cases for other AWS Services:  Transfer
        Policy: policy-tf-user-acess
        Role Name: role-tf-user-access
    ```

> Após a configuração cópie o ARN, pois iremos utilizar no próximo tópico.


## Criação do usuário 

Vamos acessar o Secret Manager, onde vamos adicionar a secret em texto, segue abaixo exemplo de configuração:

1. Secrets Manager > Create Secret > Secret Type: Other type of secret > Adicione o seguinte texto e edite conforme preferir:
```
{
  "Password": "minha-senha-123",
  "Role": "arn:aws:iam::<account-id>:role/role-tf-user-acess",
  "HomeDirectory": "bucket_name/diretório-que-vc-quer-criar-no-bucket"
}
```
2. Em Configure Secret, o nome do segredo tem que ser o id do transfer faimly e o usuário, como o exemplo abaixo:

```
s-d3cf65819e2c47a6b/beatriz
```

> O restante da configuração pode ser deixada como default



## Validando se deu tudo certo:

Acesse o terminal da sua máquina e rode o comando abaixo:

```
sftp -i transfer-key  USER_NAME@ENDPOINT_TRANSFER_FAMILY
```