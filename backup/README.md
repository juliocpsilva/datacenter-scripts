# infra-scripts

Repositório de scripts de automação para operações de infraestrutura.  
Mantido pela equipe técnica. Dúvidas: fale com o responsável técnico.

---

## Estrutura do repositório

```
infra-scripts/
└── backup/
    └── linux/
        └── dump-postgresql.sh     # Dump do PostgreSQL em servidores Linux
```

---

## Como usar os scripts

### Linux (via SSH)

Conecte no servidor como `root` e execute os passos abaixo.

**1. Baixar o script**

```bash
curl -O https://raw.githubusercontent.com/juliocpsilva/infra-scripts/main/backup/linux/dump-postgresql.sh
```

**2. Dar permissão de execução**

```bash
chmod +x dump-postgresql.sh
```

**3. Editar as configurações** (ver seção de cada script abaixo)

```bash
nano dump-postgresql.sh
```

**4. Executar**

```bash
bash dump-postgresql.sh
```

---

## Scripts disponíveis

---

### `backup/linux/dump-postgresql.sh`

Realiza o dump completo de um banco de dados PostgreSQL em servidores Linux.  
Compacta o arquivo gerado, registra o backup na tabela do banco e remove backups e logs com mais de 7 dias.

**Pré-requisitos**

- Sistema operacional: Linux
- Usuário: `root`
- PostgreSQL instalado (versão 14, path: `/usr/pgsql-14/bin`)
- Arquivo `.pgpass` configurado em `/root/.pgpass` com permissão `600`

**Configurar o `.pgpass`**

O arquivo `.pgpass` armazena a senha do banco de forma segura, evitando que ela fique exposta no script.

```bash
# Criar o arquivo
echo "localhost:PORTA:NOME_DO_BANCO:postgres:SENHA_AQUI" > /root/.pgpass

# Ajustar a permissão (obrigatório)
chmod 600 /root/.pgpass
```

Exemplo preenchido:

```
localhost:5432:meu_banco:postgres:minha_senha_segura
```

> A porta padrão do PostgreSQL é `5432`. Se o seu servidor usar uma porta diferente, ajuste conforme necessário.

**Variáveis que precisam ser ajustadas antes de rodar**

Abra o script com `nano dump-postgresql.sh` e edite o bloco de configurações no topo:

| Variável | O que é | Exemplo |
|---|---|---|
| `PG_USER` | Usuário do PostgreSQL | `postgres` |
| `PG_PORT` | Porta do PostgreSQL | `5432` |
| `PG_BIN` | Caminho dos binários do PostgreSQL | `/usr/pgsql-14/bin` |
| `NOME` | Nome do banco de dados | `meu_banco` |
| `PATH_BK` | Pasta onde os backups serão salvos | `/mnt/backup` |
| `LOG_DIR` | Pasta onde os logs serão salvos | `/mnt/backup/log` |
| `RETENCAO_DIAS` | Quantos dias manter os backups | `7` |

> Certifique-se que o usuário `root` tem permissão de escrita nas pastas `PATH_BK` e `LOG_DIR`.

**Executar manualmente**

```bash
bash dump-postgresql.sh
```

**Agendar execução automática com cron**

Para rodar todos os dias às 02:00:

```bash
crontab -e
```

Adiciona a linha abaixo, ajustando o caminho onde o script foi salvo:

```
0 2 * * * /bin/bash /caminho/para/dump-postgresql.sh
```

**O que o script faz (passo a passo)**

1. Verifica se todas as dependências estão instaladas (`pg_dump`, `psql`, `tar`, etc.)
2. Verifica se o arquivo `.pgpass` existe e tem permissão correta (`600`)
3. Cria um lockfile para evitar execuções simultâneas
4. Cria as pastas de backup e log se não existirem
5. Executa o `pg_dump` e salva o arquivo com extensão `.sfwn`
6. Compacta o arquivo em `.sfwn.tar.gz`
7. Remove o arquivo `.sfwn` intermediário
8. Remove backups com mais de `RETENCAO_DIAS` dias
9. Registra o backup na tabela `backup` do banco de dados
10. Remove logs com mais de `RETENCAO_DIAS` dias

**Verificar se o backup foi gerado**

Substitua `/mnt/backup` pelo caminho configurado na variável `PATH_BK`:

```bash
ls -lh /mnt/backup
```

**Verificar o log da última execução**

Substitua `/mnt/backup/log` pelo caminho configurado na variável `LOG_DIR`:

```bash
ls -lh /mnt/backup/log
cat /mnt/backup/log/NOME_DO_LOG.log
```

**Erros mais comuns**

| Erro | Causa | Solução |
|---|---|---|
| `ERRO FATAL: Arquivo .pgpass não encontrado` | `.pgpass` não existe | Criar conforme instruções acima |
| `ERRO FATAL: .pgpass deve ter permissão 600` | Permissão incorreta | `chmod 600 /root/.pgpass` |
| `ERRO: Backup já está em execução` | Script rodando em paralelo | Aguardar ou remover `/tmp/bk_vr.lock` |
| `Falha ao executar pg_dump` | Credenciais ou porta erradas | Conferir `.pgpass` e variáveis no script |
| `Arquivo de dump está vazio` | Banco inacessível | Verificar se o PostgreSQL está rodando |

---

## Boas práticas

- Nunca coloque senhas diretamente no script — use sempre o `.pgpass`
- Sempre teste o script manualmente antes de agendar no cron
- Confira o log após a primeira execução para garantir que tudo correu bem
- Em caso de dúvida, acione o responsável técnico antes de rodar em produção

---

## Contribuindo com novos scripts

1. Crie o script na pasta correta (`backup/linux/`, `setup/linux/`, etc.)
2. Teste localmente antes de subir
3. Atualize este README com a documentação do novo script
4. Faça o commit e push:

```bash
git add .
git commit -m "Adiciona script de [descrição]"
git push
```
