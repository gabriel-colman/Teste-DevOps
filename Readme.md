# Projeto: Configuração de Servidor Icecast com Docker, Nginx e Cliente BUTT

Este projeto tem como objetivo configurar um ambiente de transmissão de áudio usando o servidor Icecast, um proxy reverso Nginx, e o cliente de transmissão BUTT. O ambiente é configurado usando containers Docker para facilitar a implantação e a portabilidade.

## Requisitos

- Docker e Docker Compose instalados na sua máquina local.
- Cliente de transmissão de áudio BUTT instalado.

## Passo a Passo

### 1. Preparação do Ambiente

#### 1.1 Instalação do Docker

Certifique-se de que o Docker está instalado em sua máquina. Para instruções de instalação, consulte o [site oficial do Docker](https://docs.docker.com/get-docker/).

### 2. Configuração do Servidor Icecast

#### 2.1 Criar o Dockerfile para Icecast

Crie um arquivo chamado `Dockerfile` no diretório do projeto com o seguinte conteúdo:

```Dockerfile
# Usar a imagem Docker do Icecast
FROM thespider/icecast

# Expor a porta padrão do Icecast
EXPOSE 8000
```

#### 2.2 Construir e Executar o Container Icecast

Execute os seguintes comandos no terminal para construir a imagem e iniciar o container:

```bash
# Construir a imagem Docker
docker build -t meu-icecast .

# Iniciar o container Icecast
docker run -d --name icecast-container -p 8000:8000 meu-icecast
```

### 3. Configuração do Nginx como Proxy Reverso

#### 3.1 Criar o Dockerfile para Nginx

Crie um arquivo chamado Dockerfile.nginx com o seguinte conteúdo:

```Dockerfile
# Usar a imagem oficial do Nginx
FROM nginx

# Copiar o arquivo de configuração do Nginx para o container
COPY nginx.conf /etc/nginx/nginx.conf
```

#### 3.2 Criar o Arquivo de Configuração do Nginx

Crie um arquivo chamado nginx.conf no diretório do projeto com a seguinte configuração:

```nginx
events {
    worker_connections 1024;
}

http {
    server {
        listen 80;

        location / {
            proxy_pass http://icecast-container:8000;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto $scheme;
        }
    }
}
```

#### 3.3 Construir e Executar o Container Nginx

Execute os comandos abaixo para construir a imagem e iniciar o container Nginx:

```bash
# Construir a imagem Docker para Nginx
docker build -t meu-nginx -f Dockerfile.nginx .

# Iniciar o container Nginx
docker run -d --name nginx-container -p 80:80 --link icecast-container:icecast-container meu-nginx
```

### 4. Configuração do Cliente de Transmissão BUTT

#### 4.1 Instalar o BUTT

Baixe e instale o cliente BUTT a partir do site oficial.

#### 4.2 Configurar o Servidor no BUTT

Abra o BUTT e vá para Configurações > Servidor > Adicionar.
Preencha os campos conforme abaixo:
Tipo: Icecast
Endereço: localhost
Porta: 8000
Senha: hackme (ou outra senha configurada no icecast.xml)
Montagem: /stream
Clique em Adicionar para salvar.

#### 4.3 Iniciar a Transmissão

Na interface principal do BUTT, clique em Play para iniciar a transmissão.

### 5. Verificar a Transmissão no Icecast

Acesse http://localhost:8000/status.xsl no navegador.
Verifique o ponto de montagem /stream para confirmar que o stream está ativo.

### 6. Alterações no Arquivo icecast.xml

Caso precise alterar a senha ou outras configurações, siga os passos abaixo:

#### 6.1 Acessar o Container Icecast

```bash
docker exec -it icecast-container /bin/sh
```

#### 6.2 Editar o Arquivo icecast.xml

Use o editor vi para editar o arquivo /etc/icecast.xml:

```bash
vi /etc/icecast.xml
```

Localize a linha `<source-password>hackme</source-password>` e altere conforme necessário.

#### 6.3 Reiniciar o Container Icecast

Após fazer as alterações:

```bash
docker restart icecast-container
```

### 7. Testar e Confirmar a Transmissão

Clique em Play no BUTT.
Verifique novamente em http://localhost:8000/status.xsl se o stream está ativo.

Recursos

- [Documentação Icecast](https://icecast.org/documentation/)
- [Docker Hub - Icecast Image](https://hub.docker.com/r/thespider/icecast)
- [Docker Hub - Nginx Image](https://hub.docker.com/_/nginx)
- [BUTT Website](https://danielnoethen.de/butt/)

Conclusão

Este projeto demonstra como configurar um servidor de transmissão de áudio utilizando Docker, Icecast, Nginx e BUTT. A configuração e a documentação detalhada devem permitir replicar o ambiente facilmente e iniciar a transmissão de áudio ao vivo.

Nota: Certifique-se de ajustar todas as configurações conforme suas necessidades e ambiente específicos.

### Como Usar

1. Copie todo o conteúdo acima para o seu arquivo `README.md`.
2. Personalize conforme necessário, adicionando detalhes específicos do seu projeto ou ambiente.
3. Suba o `README.md` para o repositório GitHub junto com o código e os arquivos de configuração para documentar o seu projeto.

Se precisar de mais alguma modificação ou ajuda, é só avisar!