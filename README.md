# VaultID - CESS 

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

Os seguintes parâmetros são definidos dentro do arquivo prod-compose.yaml.

* **APACHE_PORT** - Define a porta exposta pela aplicação. Aceita valores entre 1024 e 65535.

* **HOSTNAME** - O nome do servidor identificado ao Apache. Utilize uma string sem espaços.

* **APACHE_SSL** 
   - Defina para true se deseja que o Apache do container forneça o serviço com TLS ativo.  
   - Espera-se que o certificado digital e a respectiva chave sejam fornecidos através de um ponto de montagem no 
   container. Descomente as respectivas referências na sessão 'volumes' e configure os arquivos conforme orientação.

        - Arquivo **path_crt** 
            - Espera-se um arquivo com a parte pública do certificado digital concatenado com as cadeias intermediárias 
            da Autoridade Certificado emissora. Todos os certificados devem estar no formato PEM (codificado em base64).

        - Arquivo **path_key** 
            - Espera-se um arquivo contendo apenas a chave privada correspondente ao certificado digital utilizado. 
            A chave privada não pode ter senha e deve estar no formato PEM (codificada em base64).


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

2 - Iniciando a aplicação.

```bash
docker-compose -f cess-compose.yaml -up -d
```

3 - Verificando o estado da aplicação:

```bash
docker ps 
```

4 - Testando a aplicação:
```bash
TESTE 
```
