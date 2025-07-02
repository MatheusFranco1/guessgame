# Jogo de Adivinhação com Flask, React, PostgreSQL e Docker

Este projeto é um jogo de adivinhação de palavras, composto por backend Flask, frontend React, banco de dados PostgreSQL e NGINX como proxy reverso e balanceador de carga. Todos os serviços são orquestrados via Docker Compose.

---

## Opções de Design

### Serviços

- **backend / backend2:**  
  Dois containers Flask para garantir alta disponibilidade e balanceamento de carga. O backend é responsável pela lógica do jogo e comunicação com o banco de dados.

- **frontend:**  
  Aplicação React servida via NGINX, responsável pela interface do usuário.

- **db:**  
  Banco de dados PostgreSQL para persistência dos dados do jogo.

- **nginx:**  
  Proxy reverso e balanceador de carga.
    - Roteia `/api/*` para os backends (com balanceamento).
    - Roteia `/` para o frontend.

### Volumes

- **db_data:**  
  Volume Docker persistente para garantir que os dados do banco não sejam perdidos ao reiniciar ou atualizar o container do banco.

### Redes

- Todos os serviços estão na mesma rede padrão do Docker Compose, permitindo comunicação interna segura e eficiente.

### Estratégia de Balanceamento de Carga

- O NGINX utiliza o bloco `upstream` para distribuir as requisições `/api/` entre os containers `backend` e `backend2`, aumentando a resiliência e escalabilidade do backend.

---

## Instruções de Instalação e Execução

### Pré-requisitos

- Docker e Docker Compose instalados

### Passos

1. **Clone o repositório:**
   ```sh
   git clone https://github.com/fams/guess_game.git
   cd guess_game
   ```

2. **Suba todos os serviços:**
   ```sh
   docker-compose up --build
   ```

3. **Acesse a aplicação:**
   - No navegador, acesse: [http://localhost:8081](http://localhost:8081)

---

## Como Atualizar os Serviços

### Atualizar o Backend

1. Faça alterações no código do backend.
2. Rebuild apenas o backend:
   ```sh
   docker-compose build backend backend2
   docker-compose up -d backend backend2
   ```

### Atualizar o Frontend

1. Faça alterações no código do frontend.
2. Rebuild apenas o frontend:
   ```sh
   docker-compose build frontend
   docker-compose up -d frontend
   ```

### Atualizar o Banco de Dados

1. Para atualizar a versão do PostgreSQL, altere a linha da imagem no `docker-compose.yaml`:
   ```yaml
   image: postgres:15
   ```
   (Troque `15` pela versão desejada)
2. Rode:
   ```sh
   docker-compose up -d db
   ```

---

## Trocar Porta ou URL

- Para rodar em outra porta, altere a linha:
  ```yaml
  ports:
    - "8081:80"
  ```
  Exemplo para porta 3000:
  ```yaml
  ports:
    - "3000:80"
  ```
  Acesse então [http://localhost:3000](http://localhost:3000).

- Para acessar por um nome personalizado (ex: `meujogo.local`), adicione `127.0.0.1 meujogo.local` ao arquivo `hosts` do seu sistema e acesse [http://meujogo.local:8081](http://meujogo.local:8081).

---

## Como Jogar

### 1. Criar um novo jogo

Acesse a url do frontend [http://localhost:8081](http://localhost:8081)

Digite uma frase secreta, envie e salve o game-id retornado.

### 2. Adivinhar a senha

Acesse novamente o frontend, vá para o endpoint breaker, entre com o game_id gerado e tente adivinhar.

---

## Estrutura do Código

### Rotas:

- **`/api/create`**: Cria um novo jogo. Armazena a senha codificada em base64 e retorna um `game_id`.
- **`/api/guess/<game_id>`**: Permite ao usuário adivinhar a senha. Compara a adivinhação com a senha armazenada e retorna o resultado.

### Classes Importantes:

- **`Guess`**: Classe responsável por gerenciar a lógica de comparação entre a senha e a tentativa do jogador.
- **`WrongAttempt`**: Exceção personalizada que é levantada quando a tentativa está incorreta.

---

## Decisões de Design

- **Balanceamento de carga:**  
  Dois containers backend para garantir disponibilidade e escalabilidade.
- **Proxy reverso:**  
  NGINX centraliza o acesso, simplificando o frontend e protegendo o backend.
- **Volumes:**  
  Persistência dos dados do banco, facilitando backup e atualização.
- **Facilidade de atualização:**  
  Cada serviço pode ser atualizado individualmente sem afetar os demais.

---

## Variáveis de Ambiente do Banco de Dados

Você pode customizar facilmente as credenciais e configurações do banco de dados PostgreSQL alterando as variáveis no serviço `db` e nos serviços backend/backend2 do `docker-compose.yaml`:

### Exemplo para o serviço `db`:

```yaml
db:
  image: postgres:15
  environment:
    - POSTGRES_USER=usuario_desejado
    - POSTGRES_PASSWORD=senha_desejada
    - POSTGRES_DB=nome_do_banco
  volumes:
    - db_data:/var/lib/postgresql/data
  restart: always
```

### Exemplo para os serviços `backend` e `backend2`:

```yaml
backend:
  environment:
    - FLASK_ENV=production
    - FLASK_DATABASE_URL=postgresql://usuario_desejado:senha_desejada@db:5432/nome_do_banco
    - FLASK_DB_TYPE=postgres
    - FLASK_DB_NAME=nome_do_banco
    - FLASK_DB_USER=usuario_desejado
    - FLASK_DB_PASSWORD=senha_desejada
    - FLASK_DB_HOST=db
    # - FLASK_DB_PORT=5432  # Opcional: só adicione se quiser usar uma porta diferente da padrão
```

**Importante:**  

Sempre mantenha os valores das variáveis do backend compatíveis com os do serviço `db`.

---

## Por que usamos Gunicorn no Backend?

O backend Flask é executado com o servidor Gunicorn em produção por vários motivos:

- **Desempenho e Confiabilidade:**  
O Gunicorn é um servidor WSGI robusto, capaz de gerenciar múltiplos processos e conexões, garantindo maior desempenho e estabilidade.

- **Multiprocessamento:**  
  Gunicorn permite rodar múltiplos workers, aproveitando melhor os recursos do servidor e aumentando a capacidade de atendimento do backend.

- **Integração com Docker e NGINX:**  
  O Gunicorn funciona perfeitamente em containers Docker e atrás de proxies reversos como o NGINX, facilitando o balanceamento de carga e a escalabilidade do sistema.

---

## Agradecimento

Obrigado por utilizar e testar este projeto! Qualquer sugestão ou contribuição é bem-vinda.


