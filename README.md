# envswitch

Alterna blocos e variáveis em arquivos `.env` usando marcadores explícitos. Útil para
trocar rapidamente entre configurações (ex.: DB de produção ↔ local) ou ligar/desligar
flags como logs e debug sem editar o arquivo manualmente.

Compatível com macOS (bash 3.2) e Linux. Sem dependências além de `bash` e `awk`.

## Instalação

```bash
curl -fsSL https://raw.githubusercontent.com/ronaldoflima/envswitch/main/envswitch \
  -o /usr/local/bin/envswitch
chmod +x /usr/local/bin/envswitch
```

Ou clone e crie um symlink onde preferir:

```bash
git clone https://github.com/ronaldoflima/envswitch.git
ln -s "$PWD/envswitch/envswitch" /usr/local/bin/envswitch
```

## Marcadores

### 1. Block switch

Alterna entre blocos completos. O bloco escolhido fica descomentado; os demais ficam
comentados com `# ` em todas as linhas.

```env
# envswitch:start prd
DB_HOST=host.docker.internal
DB_PORT=41117
DB_DATABASE=production
# envswitch:end

# envswitch:start dev
DB_HOST=pgsql
DB_PORT=5432
DB_DATABASE=local
# envswitch:end
```

### 2. Toggle

Alterna o valor de uma única variável. O marcador vai na linha **acima** da env.

```env
# envswitch:toggle querylog
QUERY_LOG=false

# envswitch:toggle loglevel debug|info|warning|error
LOG_LEVEL=debug
```

Sem lista de valores explícita, o default é `true|false`.

## Uso

```bash
envswitch                       # alterna o primeiro bloco disponível
envswitch <alias>               # bloco: ativa | toggle: cicla para o próximo valor
envswitch <alias>=<valor>       # toggle: define o valor explicitamente
envswitch -it                   # menu interativo com setas ↑↓
envswitch --list                # lista blocos e toggles encontrados
envswitch -f <arquivo>          # opera em outro arquivo (default: ./.env)
envswitch -h                    # ajuda
```

### Modo interativo

`envswitch -it` abre um menu navegável:

- ↑ / ↓ (ou `k` / `j`) para navegar
- **Enter** confirma
- `q` ou **Esc** cancela

Selecionar um bloco ativa-o. Selecionar um toggle cicla automaticamente para o
próximo valor da lista (não abre segundo menu).

Sem TTY (uso via pipe/CI), faz fallback para menu numerado.

## Exemplos

```bash
# Trocar do DB local para produção
envswitch prd

# Ligar query log
envswitch querylog

# Definir nível de log direto
envswitch loglevel=warning

# Ver o estado atual
envswitch --list
```

## Como funciona

- Blocos: cada linha entre `# envswitch:start <alias>` e `# envswitch:end` é
  descomentada (no bloco alvo) ou comentada com `# ` (nos demais). Os marcadores
  nunca são alterados.
- Toggles: o script identifica a primeira linha `KEY=VALUE` logo abaixo do marcador
  e substitui apenas o `VALUE`. Indentação é preservada.

## Licença

MIT
