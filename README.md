<p align="center">
  <img src="/images/vaultID.png"/>
</p>

# CESS
### Pré-requisitos

Para a configuração do endpoint, utilizamos como base o Docker CE, um gerenciador de containers que permite simplificar a 
configuração e gestão da solução.   
Consulte a [documentação oficial](https://www.docker.com/) para mais informações sobre 
o Docker e seu funcionamento. 

O mínimo necessário para execução:  
    	 
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

A solução não inicia acessos à rede interna. O tráfego de saída pode ser limitado aos clientes que consomem os serviços e aos endpoints citados.
    
### Configurações

Os seguintes parâmetros devem ser definidos dentro do arquivo prod-compose.yaml.

* **vaultCloudClientId** - ID da aplicação cadastrada na cloud para o CESS. Cada instância deve ter sua própria identificação.

* **vaultCloudClientSecret** - Senha da aplicação cadastrada na cloud para o CESS.

* **lifetime** - Tempo (segundos) que os arquivos (tcn) irão ficar armazenado no CESS.

* **sleep** - Intervalo (segundos) que o garbage collector irá executar para limpar os arquivos.

* **limit** - Quantidade máxima de arquivos que serão apagados a cada iteração do garbage.

* **cessUrl** - Define a URL utilizada para conexão ao Cess. Normalmente cria-se um registro de DNS apontando para o 
container.

* **redisCluster** Caso queria utilizar redis cluster, deverá setar a variavel. Obs: setar apenas se for utilizar redis cluster

* **seedsCluster** Caso habilite variável 'redisCluster', deverá configurar esta variável com os nós. Ex: ["ip1:port1", "ip2:port2"... , "ipn:portn"]

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



--- 
### Configuração do cluster (Redis) - Docker

> Nesta etapa já pressupõe que o cliente tenha 3 máquinas distintas com o docker instalado.

Será instalado 2 redis (master e slave) em cada máquina.

Como exemplo utilizaremos 3 máquinas com os respectivos IPs: <i> 192.168.0.16, 192.168.0.19 e 192.168.0.20:  </i>

Roda os seguintes comandos nos respectivos IPs

<br>

> IP [192.168.0.16]

<i>$ docker run -d --name redis-7000 -p 7000:7000 --network host redis:alpine redis-server --port 7000 --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes </i>

<i>$ docker run -d --name redis-7001 -p 7001:7001 --network host redis:alpine redis-server --port 7001 --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes </i>



> IP [192.168.0.19]


<i>$ docker run -d --name redis-7002 -p 7002:7002 --network host redis:alpine redis-server --port 7002 --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes</i>

<i>$ docker run -d --name redis-7003 -p 7003:7003 --network host redis:alpine redis-server --port 7003 --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes</i>


> IP [192.168.0.20]

<i>$ docker run -d --name redis-7004 -p 7004:7004 --network host redis:alpine redis-server --port 7004 --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes</i>

<i>$ docker run -d --name redis-7005 -p 7005:7005 --network host redis:alpine redis-server --port 7005 --cluster-enabled yes --cluster-config-file nodes.conf --cluster-node-timeout 5000 --appendonly yes</i>


E por fim execute o seguinte comando na máquina 1 (192.168.0.16): <i>$ redis-cli --cluster create 192.168.0.16:7000 192.168.0.16:7001 192.168.0.19:7002 192.168.0.19:7003 192.168.0.20:7004 192.168.0.20:7005 --cluster-replicas 1 </i>

Este comando irá criar um master e um slave para cada máquina


Para testar o cluster, acesse o primeiro redis:

$ redis-cli -c -h 192.168.0.16 -p 7000 </br>
$ set chave valor

Acesse algum outro redis:

$ redis-cli -c -p 192.168.0.20 -p 7005 </br>
$ get chave

É necessário retornar <i>"valor"</i>

---
