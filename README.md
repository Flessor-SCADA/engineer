# Flessor Engineer — skill de agente para o Flessor Studio

Skill que permite a um agente de IA (Claude Code, Codex e similares) **instalado na mesma máquina do Flessor Studio** consultar e editar o projeto aberto: menus, telas, máquinas, mapas de variáveis, alarmes, tags virtuais e relatórios personalizados — sem automação de interface e sem código-fonte. Flessor é a linha de produtos da Flessak Eletro Industrial S.A.

**Onde funciona:** agentes instalados no Windows da máquina do Studio, como Claude Code e Codex. Não funciona em chat na web ou no celular.

> **Requisito: Flessor Studio versão 6 ou superior.** Versões anteriores não têm a ponte. Confira a versão em **AJUDA → Sobre**.

## Como funciona

1. No Flessor Studio, abra **EDITAR → Janelas → Conectar Agente** (indisponível em instância somente consulta). O diálogo mostra um código de 6 caracteres (o *handle* daquela instância) e fecha sozinho quando o agente se conecta.
2. Peça ao agente o que quiser fazer no projeto e informe o código quando ele pedir.
3. O agente troca arquivos com o Studio numa pasta temporária do seu usuário do Windows. A ponte não abre porta de rede.
4. O que o agente altera no projeto fica **em memória** até você salvar no Studio ou confirmar um registro de versão, que também pode gravar o projeto. Revise antes. Ao salvar, o Studio pode dizer que "não tem alterações" e perguntar se salva mesmo assim: **responda Sim**. Fechar sem salvar perde tudo, sem aviso.

Para dados de operação (tempo real, histórico, alarmes reais), use o plugin irmão **[flessor-analytics](https://github.com/Flessor-SCADA/analytics)**.

## Exemplos de pedido

Troque pelos nomes do seu projeto:

- "Liste as máquinas e os menus deste projeto."
- "Explique o que mostra a tela Vertedouro."
- "Crie uma máquina Modbus para este medidor, com mapa, alarme de falha de comunicação e uma tela simples de leitura."

## Instalação

### Claude Code (plugin)

```text
/plugin marketplace add Flessor-SCADA/engineer
/plugin install flessor-engineer@flessor-engineer
```

O formato é `plugin@marketplace`, e aqui os dois têm o mesmo nome. O repositório se chama `engineer` porque fica na organização `Flessor-SCADA`.

Depois, é só pedir em linguagem natural ("crie uma tela para o medidor X"). A skill se chama `flessor-engineer:flessor-studio-bridge` e também pode ser chamada por esse nome. Se ela não aparecer, reinicie o Claude Code e confira com `/plugin`.

### Codex

Baixe este repositório (**Code → Download ZIP** ou `git clone`) e copie a pasta `skills\flessor-studio-bridge` para `%USERPROFILE%\.codex\skills\` (crie a pasta se não existir) ou para `.agents\skills\` da pasta onde você abre o Codex.

### Outros agentes / instalação manual

Copie `skills\flessor-studio-bridge` para a pasta de skills do seu agente (no Claude Code: `%USERPROFILE%\.claude\skills\`). A skill é Markdown puro, no formato aberto `SKILL.md`.

### Já instalou a versão 1.0?

O nome mudou de `engineer` / `studio-agent-bridge` para `flessor-engineer` / `flessor-studio-bridge`.

- **Plugin:** rode `/plugin uninstall engineer@flessor-engineer`, depois `/plugin marketplace update flessor-engineer` e instale de novo.
- **Cópia manual:** apague a pasta `studio-agent-bridge` e copie a nova.

## Segurança — leia antes de usar

- **A ponte fica ligada até você fechar o Studio.** Ela liga quando você abre "Conectar Agente" e não desliga ao fechar o diálogo. Para desligar, feche o Studio.
- **O código de 6 caracteres serve para escolher a instância, não é senha.** Enquanto a ponte estiver ligada, qualquer programa rodando com o seu usuário do Windows pode comandar o Studio. Use a ponte só numa estação de engenharia confiável e feche o Studio quando terminar.
- **O conteúdo do projeto vai para o provedor do modelo de IA.** O que o agente lê do projeto é enviado ao serviço de IA que você usa — e o XML do projeto inclui dados de conexão, como usuário e senha do banco e credenciais de equipamentos. Use somente um provedor aprovado pela sua empresa para esse tipo de dado.
- **Pontos de atenção redobrada** (a skill instrui o agente a avisar você antes):
  - **tag virtual**: depois de salva, a fórmula roda sozinha no runtime de produção — em SQL, consulta o banco; em JScript, executa código no servidor;
  - **expressões em geral**: condições e fórmulas salvas no projeto podem ser avaliadas no servidor. Revise como revisa código;
  - **controle `fCustom`**: o HTML/JS roda no navegador de cada operador que abrir a tela.
- **Arquivos em disco.** Respostas e requests que falharam, que podem conter o XML do projeto com credenciais, ficam na pasta temporária do seu usuário até o Studio fechar normalmente. O histórico do projeto guarda o XML como está.
- **Confirmações são do agente, não do produto.** A skill instrui o agente a pedir sua confirmação antes de registrar uma versão no histórico do projeto (`commit-projeto`); o Studio não exige essa confirmação, e a versão fica registrada com o seu usuário.

## Problemas e sugestões

Falha de segurança: veja [SECURITY.md](SECURITY.md). O resto: abra uma issue em <https://github.com/Flessor-SCADA/engineer/issues>. Não inclua senhas, IPs nem dados do seu projeto.

## Licença

Uso conforme o arquivo [LICENSE](LICENSE). Fabricante: Flessak Eletro Industrial S.A.
