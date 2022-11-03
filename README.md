# Terraform
Estudo sobre a ferramenta open source Terraform, responsável pela geração automática da infraestrutura de um projeto.

## Instalação no Windows
link: https://www.terraform.io/downloads
> Após o download do arquivo binário do Terraform, incluir o caminho do arquivo no PATH do Sistema Operacional.

## Instalação no Linux (Ubuntu/Debian)
Digitar os seguintes comandos (conforme documentação oficial do Terraform):
```
wget -O- https://apt.releases.hashicorp.com/gpg | gpg --dearmor | sudo tee /usr/share/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/hashicorp.list
sudo apt update && sudo apt install terraform
```

Após a instalação, digitar o comando **terraform** e **terraform -v** no terminal para testar a instalação.

## Terraform Language
Linguagem para descrever toda a infraestrutura desejada que será criada pelo Terraform.

O VS Code fornece uma extensão para *Sintax Highlighting* e *Autocompletion* para o Terraform, criada pela própria **HashiCorp**:

![image](https://user-images.githubusercontent.com/39681960/199621178-a033f8fb-653b-4176-94f9-e010bf0dbc17.png)

Ao criar um novo arquivo no VS Code (**CTRL + N**), escolha a linguagem Terraform que já terá o suporte da nova extensão ou simplesmente nomeie o arquivo com a **extensão .tf**

### Comandos
**terraform init**: inicializa o diretório de trabalho para começar a trabalhar com o Terraform. (semelhante ao git init)

## Terraform Registry
Vamos utilizar o provider oficial para a AWS, seguindo a documentação disponível no site do Terraform:
https://registry.terraform.io/providers/hashicorp/aws/latest/docs

Para autenticação e configuração dentro da AWS, temos algumas opções para acesso:
- Parameters in the provider configuration
- Environment variables
- Shared credentials files
- Shared configuration files
- Container credentials
- Instance profile credentials and region

Obs: A opção padrão de configuração das credenciais é dado no botão **Use Provider** no site da documentação:

![image](https://user-images.githubusercontent.com/39681960/199625167-5593c499-ece7-4135-916c-2567609cf00e.png)

```
 terraform { 
   required_providers { 
     aws = { 
       source = "hashicorp/aws" 
       version = "4.37.0" 
     } 
   } 
 } 
```
```
 provider "aws" { 
    /*Configuration options*/ 
 } 
```

Após a inclusão do código acima no arquivo **main.tf**, é necessário executar o comando **terraform init** sempre que uma modificação no projeto for feita. Executado o comando, o Terraform fará a instalação do **provider hashicorp/aws v4.37.0**, criando uma **pasta terraform** para o provider e um **arquivo .terraform.lock.hcl** que grava as opções do provider utilizado. 
> Obs: a pasta **terraform** deve ser colocada no arquivo **.gitignore** mas o arquivo **.terraform.loc.hcl** deve ser enviado ao repositório para garantir que o Terraform faça sempre a configuração correta quando o comando **terraform init** for executado.

###  AWS CLI 
É uma Command Line Interface utilizada para o gerenciamento dos recursos na AWS.

- Instalação no Windows
Link: https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html
1. Baixar o arquivo msi (instalador do AWS CLI);
2. Execute o instalador e apenas clique em Next até o final;
3. Se necessário, reinicie o VS Code para que a nova variável de ambiente do AWS CLI seja reconhecida.

- Instalação no Linux
```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
sudo ./aws/install
aws --version
```

Obs: **executar um comando de cada vez pois, por exemplo, a descompactação do arquivo awscliv2.zip exige interação com o terminal.**

### Geração do arquivo de Credenciais
1. Acessar a sua conta da **AWS > Security Credentials > Access Keys > Create New Access Key**
2. Digitar o comando **aws configure** no **terminal**
3. Será solicitada sua **Access Key Id** gerada na **AWS**
4. Será solicitada a sua **Secret Access Key** também gerada na **AWS**
5. Será solicitada sua **região padrão** da AWS (código da região): **sa-east-1** (São Paulo, por exemplo)
6. Será solicitado o **formato de saída**: escolha **json**, por exemplo

Ao final, será criada uma **subpasta oculta .aws** dentro da pasta do usuário logado no sistema (**/home/<nome_usuario**) com os **arquivos de credenciais e configurações** do **AWS CLI**.

Obs: os passos realizados acima servem para preencher o bloco que foi apresentado no início desse documento, relacionado à configuração do provider:
```
 provider "aws" { 
    /*Configuration options*/ 
 } 
```
Poderia ser incluído o caminho dos arquivos gerados nessa configuração, conforme mostrado na documentação:
```
provider "aws" {
  shared_config_files      = ["/Users/tf_user/.aws/conf"]
  shared_credentials_files = ["/Users/tf_user/.aws/creds"]
  profile                  = "customprofile"
}
```
Há quem diga que por ser o comportamento padrão, o código acima não precisa ser explicitamente informado. No meu caso, precisei informar o caminho dos arquivos e removi a chave profile para que o plano de execução (que será visto mais à frente) fosse executado:
```
provider "aws" {
  shared_config_files      = ["/<caminho>/.aws/config"]
  shared_credentials_files = ["/<caminho>/.aws/credentials"]
}
```

## Criação de uma instancia Windows (Free Tier)
Link para a documentação de referência: https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance
No link acima, é apresentado um exemplo de código do Terraform para a criação de uma instância na AWS:
```
resource "aws_instance" "this" {
  ami                       = "ami-0dcc1e21636832c5d"
  instance_type             = "m5.large"
  host_resource_group_arn   = "arn:aws:resource-groups:us-west-2:012345678901:group/win-testhost"
  tenancy                   = "host"
}
```
Vamos alterar um pouco o exemplo:
```
resource "aws_instance" "Windows_instance" { # Define o recurso como uma instância da AWS e a nomeia de Windows_instance
  ami           = "ami-017cdd6dc706848b2"    # Informa a chave da imagem do sistema operacional Windows Server 2022 elegível
  instance_type = "t2.micro"                 # Tipo de instância elegível a Free Tier
  ebs_block_device {
    volume_size = "30"  # Capacidade de armazenamento do volume (30 GB é o máximo elegível para Free Tier)
    volume_type = "gp2" # Tipo de volume de armazenamento também elegível a Free Tier (EBS General Purpose)    
  }
  tags = {
    Name = "FreeTier" # Define um rótulo para a instância
  }
}
```

Criado o código para a instância, vamos utilizar dois comandos do Terraform para análise e formatação do código:
- **terraform fmt**: formata o código para o padrão do Terraform
- **terraform validate**: verifica se o código escrito contém algum erro de sintaxe

Feita a formatação e análise, podemos agora rodar o comando que mostrará o planejamento do Terraform para realizar a criação da instância de acordo com o código escrito:
- **terraform plan**: mostra todo o plano de execução que o Terraform fará para a criação da instância EC2 na AWS
- **terraform apply**: se estiver tudo certo com o planejamento, este é o comando para a criação da instância na AWS, via terraform.









