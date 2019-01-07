<p align="center">
  <img src="/images/vaultID.png"/>
</p>

# CESS
### Pré-requisitos

Para a configuração do endpoint, utilizamos como base o Docker CE, um gerenciador de containers que permite simplificar a 
configuração e gestão da solução.   
Consulte a [documentação oficial](https://www.docker.com/) para mais informações sobre 
o Docker e seu funcionamento. 

O mínimo necessário para execução em cada plataforma: 

* **Windows** - Documentação oficial [Windows](https://docs.docker.com/docker-for-windows/). 
    - Versões:  
        - Windows 10 64bit: Pro ou Enterprise
        - Windows Server 2016.
    - CPU com suporte a SLAT.
    - Flag de virtualização na BIOS do servidor.    
    - 2GB de RAM para a aplicação.
	- A quantidade de CPU depende do volume de assinaturas realizadas.       
    - 5GB de disco para aplicação e logs, sem contar o sistema operacional.
	- Acesso à internet para instalação de aplicativos.	
	
    Maiores informações: 
    - [Docker para Windows: O que saber antes de instalar](https://docs.docker.com/docker-for-windows/install/#what-to-know-before-you-install)  
    	 
* **Linux**  
Docker para [Centos](https://docs.docker.com/v17.12/install/linux/docker-ce/centos/#install-docker-ce )  
Docker para [Debian](https://docs.docker.com/v17.12/install/linux/docker-ce/debian/#install-docker-ce)
    - Versões:
        - Debian 8+ ou Centos 7+ x86_64
    - 2GB de RAM para a aplicação.
    - A quantidade de CPU depende do volume de assinaturas realizadas.       
    - 5GB de disco para aplicação e logs, sem contar o sistema operacional.
    - Acesso à internet para instalação de aplicativos.
    - Acesso por parte das aplicações integradas à aplicação.
    

**Atenção**    
Para ambas as plataformas é necessário a instalação do [Docker-compose](https://docs.docker.com/compose/install/#install-compose).      
    
### Configurações

Os seguintes parâmetros devem ser definidos dentro do arquivo prod-compose.yaml.

* **vaultCloudClientId** - ID da aplicação cadastrada na cloud para o CESS. Cada instância deve ter sua própria identificação.

* **vaultCloudClientSecret** - Senha da aplicação cadastrada na cloud para o CESS.

* **cessUrl** - Define a URL utilizada para conexão ao Cess. Normalmente cria-se um registro de DNS apontando para o 
container.

* **HOSTNAME** - O nome do servidor identificado ao Apache. Utilize uma string sem espaços.

* **APACHE_SSL** 
   - Defina para true se deseja que o Apache do container forneça o serviço com TLS ativo.  
   - Espera-se que o certificado digital e a respectiva chave sejam fornecidos através de um ponto de montagem no 
   container. Descomente a sessão 'volumes' e configure os arquivos conforme orientação.

        - Arquivo **path_crt** 
            - Espera-se um arquivo com a parte pública do certificado digital concatenado com as cadeias intermediárias 
            da Autoridade Certificado emissora. Todos os certificados devem estar no formato PEM (codificado em base64).

        - Arquivo **path_key** 
            - Espera-se um arquivo contendo apenas a chave privada correspondente ao certificado digital utilizado. 
            A chave privada não pode ter senha e deve estar no formato PEM (codificada em base64).

* **PORTA_EXTERNA** - Define a porta pela qual o serviço do CESS será exposto.

#### Exemplo:

Considerando o cenário:
    - Ativando o SSL
    - Os certificados estão salvos na pasta /opt/certs/cert.pem e /opt/certs/cert.key
    - A url de acesso externo será https://cess.vaultid.com.br
    - A porta que o container deve expor é a 443
    
Teremos a seguinte configuração:

```yaml
    ...
      # Se necessário, edite apenas as variávies abaixo: #
      - "cessUrl=https://cess.vaultid.com.br"
      - "APACHE_SSL=true"
    ports:
      # Definir a PORTA_EXTERNA pela qual o container será exposto na rede.
      - 443:8080
    volumes:
      - /opt/certs/cert.pem:/etc/apache2/cert/cert.pem
      - /opt/certs/cert.key:/etc/apache2/cert/cert.key
    ... 
```

### Executando o CESS

* **Atenção:** Antes de continuar é necessário solicitaro acesso ao repositório de imanges do CESS diretamente à equipe 
de integração da VaultID.
   
Após concluir e validar a instalação do docker e do docker-compose, salve e configure o arquivo cess-compose.yaml no servidor de escolhido.
Pela linha de comando, navegue até a pasta de destino do arquivo e execute:

1 - Docker login.  

De posse do usuário e senha fornecidos, execute:
```bash
docker login harbor.lab.vaultid.com.br
```

<p align="center">
  <img src="/images/login.png"/>
</p>

2 - Iniciando a aplicação.

```bash
docker-compose -f cess-compose.yaml up -d
```

<p align="center">
  <img src="/images/dockerup.png"/>
</p>

3 - Verificando o estado da aplicação:

```bash
docker ps 
```

<p align="center">
  <img src="/images/dockerps.png"/>
</p>

4 - Testando a aplicação:

Após a confirmação de execução da aplicação é possível validar o estado da mesma acessando a URL configurada 
em **cessUrl** ou, diretamente no servidor com a combinação **IP_SERVIDOR:APACHE_PORT**. 

<p align="center">
  <img src="/images/teste.png"/>
</p>
