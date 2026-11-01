<div align="center">

# CTF.Term

**Cheatsheets dinâmicos para laboratórios de cibersegurança e CTF.**

Uma aplicação web estática para organizar comandos, preencher variáveis do laboratório e copiar instruções para o Kali Linux — sem executar comandos no alvo.

**Português** · [English](README.en.md)

</div>

---

## Sobre o projeto

O **CTF.Term** é uma aplicação acessível pelo navegador que transforma comandos do terminal e anotações técnicas em um cheatsheet reutilizável. Em vez de editar IPs, domínios, usuários e caminhos em cada comando, você informa esses valores uma vez na aba **Variáveis** e utiliza identificadores como `{{ip}}` e `{{domain}}` nos templates.

> **Uso responsável:** o aplicativo apenas monta e copia textos. Não se conecta ao alvo, não valida permissões e não executa comandos. Utilize os exemplos somente em ambientes em que você tenha autorização.

## Funcionalidades

- **62 variáveis padrão**, organizadas em grupos: alvo/rede, web, autenticação, wordlists, Active Directory/Windows e túneis/pivoting.
- **Variáveis dinâmicas** com a sintaxe `{{nome_da_variavel}}` e visualização dos valores derivados.
- **Nmap como única categoria de comandos inicial**, com três exemplos editáveis.
- **Categorias personalizadas:** criar, editar, escolher símbolo e excluir categorias no menu lateral.
- **Comandos personalizados:** criar, editar, excluir, pesquisar e copiar, incluindo comandos com várias linhas.
- **Destaque de sintaxe** para ferramentas, parâmetros, strings, placeholders e comentários Bash iniciados por `#`.
- **Busca** de comandos na categoria atual e filtro de variáveis.
- **Armazenamento local automático**, importação/exportação de workspace em JSON e exportação de cheatsheet em Markdown.
- **Interface responsiva**, sem backend e sem dependências JavaScript externas.

## Começando

### Opção A — Acesso offline (clonando o repositório)

Clone o projeto uma vez, enquanto estiver conectado à internet:

```bash
git clone https://github.com/e-pke/ctf-term.git
cd ctf-term
```

Depois do clone, você poderá usar o CTF.Term **localmente, sem conexão com a internet**. No Kali Linux, dentro da pasta do projeto, execute:

```bash
python3 -m http.server 8000
```

Abra **http://localhost:8000** no navegador. O servidor é local e não exige Node.js, banco de dados ou `npm install`.

### Opção B — Acesso online (GitHub Pages)

Acesse o CTF.Term diretamente pelo navegador, sem baixar ou instalar nada:

```text
https://e-pke.github.io/ctf-term/
```

A versão online mantém as funcionalidades de variáveis, categorias, comandos e importação/exportação. As personalizações são salvas no `localStorage` do seu navegador, **não no repositório GitHub**, e não são sincronizadas entre dispositivos.

## Como usar

### 1. Configure as variáveis

Abra **Variáveis** no menu lateral e preencha os campos relevantes para o laboratório. Os valores exibidos como exemplos nos campos são apenas *placeholders*; eles não são preenchidos automaticamente.

| Campo | Exemplo | Uso no comando |
| --- | --- | --- |
| IP do alvo | `10.10.10.10` | `{{ip}}` |
| Domínio | `domain.htb` | `{{domain}}` |
| Portas TCP | `22,80,443` | `{{ports_tcp}}` |
| Usuário | `john` | `{{username}}` |
| Wordlist de diretórios | `/usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt` | `{{dir_wordlist}}` |
| Porta HTTP | `8080` | `{{http_port}}` |

Você também pode criar variáveis próprias com **+ Nova variável**. Os identificadores aceitam letras minúsculas, números e `_`, começando por uma letra.

### 2. Use um comando existente

A categoria inicial **Nmap** contém três comandos. Um exemplo de template:

```bash
nmap -sC -sV -p {{ports_tcp}} {{ip}}
```

Com `ports_tcp = 22,80,443` e `ip = 10.10.10.10`, o card exibirá:

```bash
nmap -sC -sV -p 22,80,443 10.10.10.10
```

Clique em **Copiar** para levar o comando preenchido à área de transferência. Se houver variáveis obrigatórias pendentes, o aplicativo informará seus nomes e bloqueará a cópia daquele card.

### 3. Crie categorias

Use **+ Nova categoria** no menu lateral para informar nome, identificador, símbolo, título e descrição. A nova categoria aparece imediatamente no menu. Em seguida, selecione-a e clique em **+ Novo comando**.

### 4. Crie ou edite comandos

Cada comando possui **categoria**, **resumo/título**, **descrição opcional** e **template**. Use as variáveis diretamente no texto:

```bash
# Descoberta de diretórios
ffuf -u 'http://{{domain}}{{http_port}}/FUZZ' -w '{{dir_wordlist}}' -ac
```

Você decide onde colocar `http://`, `https://`, aspas simples ou aspas duplas. O aplicativo faz **substituição literal**, sem adicionar aspas ou escapar caracteres especiais do shell automaticamente.

Para comandos em várias linhas, os comentários iniciados por `#` recebem destaque visual. O símbolo `$` é apenas decorativo em comandos de uma linha e **não é copiado**.

### 5. Pesquise

O campo de busca abaixo da descrição da categoria filtra os comandos pelo título, descrição e template. Na aba **Variáveis**, a busca filtra os campos pelo nome, identificador, grupo e dica. Nos comandos, use também `Ctrl+K` (ou `⌘+K`) ou `/` para focar a busca, quando não estiver digitando em outro campo.

## Variáveis automáticas e regras especiais

| Variável | Comportamento |
| --- | --- |
| `{{dc_ip}}` | Se o campo estiver vazio, usa `{{ip}}`. |
| `{{dns_server}}` | Se o campo estiver vazio, usa `{{domain}}`. É necessário que esse hostname corresponda a um servidor DNS acessível para consultas como `dig @...`. |
| `{{http_port}}` | `8080` vira `:8080`; vazia não insere nada. Não adiciona protocolo. |
| `{{base_dn}}` | Derivada de `domain.htb` como `DC=domain,DC=htb`. |
| `{{ldap_bind}}` | Derivada de usuário e domínio, como `john@domain.htb`. |

Exemplo de LDAP:

```bash
ldapsearch -x -H 'ldap://{{dc_ip}}' -D '{{ldap_bind}}' -W -b '{{base_dn}}'
```

**Observações:** os valores derivados dependem dos campos preenchidos; o campo `http_port` aceita portas válidas de `1` a `65535`. Se você usar templates de SSTI/Jinja2, lembre que expressões como `{{config}}` podem ser interpretadas como placeholders do CTF.Term; uma expressão com espaços, como `{{ config }}`, não corresponde ao padrão de substituição atual.

## Privacidade e armazenamento

O CTF.Term usa o **`localStorage` do navegador** com a chave `ctf-term:workspace:v6`. As alterações feitas na interface normalmente permanecem após recarregar a página, mas não são sincronizadas com GitHub, outras máquinas ou outros navegadores.

Os seguintes campos padrão são tratados como sensíveis e ficam **somente na memória da aba**, sem persistência ou exportação de seus valores:

`password`, `ntlm_hash`, `fake_password`, `jwt_token`, `krbtgt_hash`, `service_hash` e `snmp_community`.

Há controles para mostrar/ocultar os valores sensíveis. Para copiar um comando que dependa de um desses campos preenchidos, é preciso autorizar sua exibição na interface.

> **Importante:** variáveis personalizadas **não recebem automaticamente** essa proteção. Não salve segredos reais nelas. **Senhas, tokens ou hashes escritos literalmente no texto de um comando também serão salvos e exportados**. Além disso, comandos copiados ou executados podem expor dados no clipboard, no histórico do shell ou em registros de ferramentas.

Se você limpar os dados do site ou mudar de origem (por exemplo, de `file://` para `http://localhost:8000`, ou para o domínio do GitHub Pages), o armazenamento poderá ser diferente. Exporte seus dados antes de mudar de endereço.

## Importação, exportação e backups

| Recurso | O que faz |
| --- | --- |
| **Exportar JSON** | Baixa o workspace com variáveis não sensíveis, categorias e comandos. |
| **Importar JSON** | Substitui **todo o workspace atual** pelo conteúdo do arquivo após confirmação. |
| **Cheatsheet .md** | Exporta os comandos das categorias em Markdown, com valores não sensíveis preenchidos. |
| **Restaurar tudo** | Volta às 62 variáveis vazias e aos seis comandos Nmap padrão, excluindo as personalizações locais. |

O importador aceita o formato oficial **`ctf-term.workspace` / schema `6`**. Uma estrutura simplificada é:

```json
{
  "format": "ctf-term.workspace",
  "schema": 6,
  "workspace": {
    "variables": {},
    "customVariables": [],
    "categories": [],
    "commands": []
  }
}
```

- O arquivo JSON deve ter no máximo **1 MB** para ser importado pela interface.
- O aplicativo aceita até **60 categorias personalizadas**, **80 variáveis personalizadas** e **300 comandos** no workspace.
- **Um Markdown ou JSON de terceiros não é importável automaticamente** só por conter comandos: um JSON de cheatsheets precisa ser convertido para o esquema acima e respeitar as categorias/IDs válidos. A importação continua sendo **substitutiva**, não uma mesclagem.

**Faça backup antes de importar outro workspace, restaurar os padrões ou limpar os dados do navegador.**

## Segurança e limitações

- Os comandos são **templates educacionais**, não resultados de varreduras realizadas pelo aplicativo.
- O CTF.Term **não executa comandos**, não faz enumeração automaticamente e não verifica se uma ferramenta está instalada.
- A substituição literal não protege contra expansão de shell. Revise aspas, caracteres especiais e os valores inseridos antes de executar.
- Alguns exemplos externos podem ser payloads, entradas para consoles especializados ou trechos de configuração, não comandos Bash prontos.
- A disponibilidade de ferramentas, opções e sintaxe depende da versão instalada no Kali Linux.

## Créditos e atribuições

O CTF.Term foi desenvolvido como ferramenta de organização de cheatsheets para estudo em laboratórios autorizados.

- **Design:** interface baseada em `#212121`, com realces inspirados no tema [Nord Wave](https://cmuxthemes.com/themes/nord-wave/).
- **Fonte:** [JetBrains Mono](https://www.jetbrains.com/lp/mono/).
- **Cheatsheets opcionais:** cheatsheet personalizado baseado em [momenbasel/htb-writeups](https://github.com/momenbasel/htb-writeups).

A coleção de terceiros **não faz parte das categorias padrão** do aplicativo.

---

<div align="center">

**CTF.Term — configure uma vez, reutilize seus comandos em cada laboratório.**

[Read this documentation in English](README.en.md)

</div>
