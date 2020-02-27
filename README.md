<p align="center">
  <img src="/images/vaultID.png"/>
</p>

# CESS
### Pré-requisitos

Para a configuração do endpoint, utilizamos como base o Docker CE, um gerenciador de containers que permite simplificar a 
configuração e gestão da solução.   
Consulte a [documentação oficial](https://www.docker.com/) para mais informações sobre 
o Docker e seu funcionamento. 

O mínimo necessário para execução é:   
    	 
* **Linux**  
Docker para [Centos](https://docs.docker.com/install/linux/docker-ce/centos/)  
Docker para [Debian](https://docs.docker.com/install/linux/docker-ce/debian/)
    - Versões:
        - Debian 8+ ou Centos 7+ x86_64
        - Docker versão 18+.
    - 2GB de RAM para a aplicação.
    - A quantidade de CPU depende do volume de assinaturas realizadas.       
    - 5GB de disco para aplicação e logs, sem contar o sistema operacional.
    - Acesso à internet para instalação de aplicativos.
    - Acesso por parte das aplicações integradas à aplicação.
    
**Atenção**    
É necessário a instalação do [Docker-compose](https://docs.docker.com/compose/install/#install-compose).

**Rede e conectividade**  
Para o funcionamento do CESS é necessário acesso aos endpoints do provedor de assinaturas nas URLs:

  - apicloudid.vaultid.com.br
  - portalapicloudid.vaultid.com.br
   
Também é necessário acesso aos repositórios de LCR's (lista de certificados revogados) referentes aos certificados utilizados. Os repositórios variam de acordo com a Autoridade Certificadora responsável pela emissão do certificado.

Para clientes emitindo pela AC Soluti, os endpoints são:

  - ccd.acsoluti.com.br
  - ccd2.acsoluti.com.br
  
Para a utilização de carimbo no tempo é necessário liberar o endpoint do TSA. Esse endpoint varia de acordo com a solução escolhida.  

A comunicação entre o CESS e os endpoints citados não pode ser realizada com inspeção SSL/TLS ou algum tipo de homem do meio que intercepte e interfira na abertura de sessão.

A solução não inicia acessos à rede interna. O tráfego de saída pode ser limitado aos clientes que consomem os serviços, aos HSMs utilizados e aos endpoints citados.
    
### Configurações

Os seguintes parâmetros devem ser definidos dentro do arquivo prod-compose.yaml.

* **cessUrl** - Define a URL utilizada para conexão ao Cess. Normalmente cria-se um registro de DNS apontando para o 
container.

* **cacheDir** - Define local onde será armazenado os caches (UTILIZAR ESSE PARAMETRO APENAS QUANDO UTILIZAR CACHE FILESYSTEM)

* **HSM_LOAD_BALANCE_LIST** - Arquivo que contém a lista de ips e portas dos HSM

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

---
#### Utilizando HSM local

É possíve utilizar um HSM Dinamo localmente para armazenamento dos certificados.  
Para isso, é necessário a criação de um usuário com permissão de criar e listar slots no HSM.

Nas configurações, configure os campos:

```yaml
      - "hsmIp=IP_HSM"
      - "hsmPort=4433"
      - "zSlotManagerUser=false"
      - "zSlotManagerPw=false"
```
      
Defina o IP e porta do HSM Dinamo local. 

A utilização da API com HSM local utiliza um formato diferente para autenticação:

Utilizando HSM em nuvem:
```php
Authorization => Bearer <token de acesso do usuário>
```

Utilizando HSM local:
```php
Authorization => VCSchema <schema vault cloud encodado em base64>
```

O VCSchema pode ser definido da seguinte forma:
           
* username:password  
* username:password@ip  
* username:password@ip:port  
* username|accesstoken  
* username|accesstoken@ip  
* username|accesstoken@ip:port  

O **username** e **password** utilizado pelo VCSchema são os mesmos utilizados para autenticação direta no HSM.
Um mesmo usuário/slot pode possuir mais de um certificado, sendo que nesse caso, a aplicação deve repassar o certificate_alias 
( alias da chave dentro do slot ) como parâmetro na chamada da API caso queira utilizar um certificado específico. 
Caso contrário, o último certificado listado no slot será utilizado.

##### Configurações para HSM local

* signatureAdapter=DinamoPkcs12Adapter
    - Ativa o adapter para utilização de HSMLocal
    
* hsmIp=IP_HSM
    - IP do HSM local

* hsmPort=HSM_PORT
    - Porta do HSM local. Default para 4433

* zSlotManagerUser
    - Usuário no HSM com permissão para criar e listar slots.

* zSlotManagerPw
    - Senha do usuário citado acima.
    
Ao ativar a importação de arquivos p12, é necessário montar um volume com esses arquivos.   
No arquivo cess-compose.yaml, na seção **volumes**, subistitua a tag path pelo caminho do diretório local 
onde os certificados estão armazenados.

```yaml
    volumes:
      - path:/var/www/data
```

---

#### Exemplo:

Considerando o cenário:  
 - SSL ativo;
 - Os certificados estão salvos na pasta /opt/certs/cert.pem e /opt/certs/cert.key;  
 - A url de acesso externo será https://cess.vaultid.com.br;
 - A porta que o container deve expor é a 443;  
    
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

* **Atenção:** Antes de continuar é necessário solicitar o acesso ao repositório de imagens do CESS diretamente à equipe 
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
  <img src="/images/dockerps2.png"/>
</p>

4 - Testando a aplicação:

Após a confirmação de execução da aplicação é possível validar o estado da mesma acessando a URL configurada 
em **cessUrl** ou, diretamente no servidor com a combinação **IP_SERVIDOR:PORTA_EXTERNA**. 

<p align="center">
  <img src="/images/teste.png"/>
</p>
