# Tags virtuais e Relatório Personalizado

## Tags virtuais — KPIs derivados de uma norma, regra ou documentação

Tag virtual (`lista: "tagsvirtuais"`) deriva um valor de outras tags (inclusive de **outras máquinas**) ou busca um valor no banco por SQL. É o caminho para transformar uma norma/documentação técnica numa série de KPIs.

- **Fórmula = campo `descricao`. Nome = campo `variavel`** (convenção `.tv_NomeDaTag`). Não existe campo `formula`.
- **Dois motores, escolhidos pelo texto da fórmula:**
  - contém `SELECT` **e** `FROM` (em qualquer lugar, sem diferenciar maiúscula — inclusive dentro de string ou comentário) → **consulta SQL** ao banco do supervisório, no mínimo a cada 60 s;
  - qualquer outra coisa (de `.a + .b` a um `function(){...}` completo) → **expressão JScript.NET**. Expressão simples e JS são o mesmo motor.
- **Referência entre máquinas:** `NomeDaMaquina:.variavel` (ex.: `Central:.nivelReservatorio`). Aceita agregação encadeada, como `.MA(60)` (média móvel). `%M` = nome da própria máquina (fórmula genérica reaproveitável). **Variável da mesma máquina dona do menu dispensa o prefixo**: `.variavel` basta.
- **Bit de uma word:** `.tag[N]` (N a partir de 0) é mais simples que `(.tag & (1 << N)) == (1 << N)`. As duas formas funcionam; se o bit não bater com o esperado numa word específica, teste contra o valor real.
- **Onde fica:** num **menu**, não numa máquina. Tag de um equipamento → menu dele. Tag que agrega várias máquinas → menu "guarda-chuva" (central/KPIs).
- **Frequência:** campo `gravacao`, em segundos. A ponte **não** aceita os presets da tela ("1h", "Mensal"); converta: Instantâneo `0` (**padrão se omitir**), 1 s `1`, 10 s `10`, 30 s `30`, 1 min `60`, 15 min `900`, 30 min `1800`, 1 h `3600`, 4 h `14400`, 12 h `43200`, 24 h `86400`, Semanal (sáb) `604799`, Semanal (sex) `604798`, Mensal `2591999`, Anual `31535998`. A ponte não faz a checagem de coerência que a tela do Studio faz — aplique o bom senso (SQL a cada 1 s vira 60 s; "instantâneo" numa fórmula que deveria agregar por hora não faz sentido).
- **Criar/editar:** `update-menu-variavel` com `lista: "tagsvirtuais"`, `variavel`, `descricao`, `gravacao` (cria se não existir). Para uma série: um request por tag, em sequência; no fim, confira com `menu-variaveis` **e** `menu-xml` antes de dizer que terminou.

### Atenção séria: tag virtual é código que roda em produção

**Toda expressão salva no projeto pode ser avaliada no servidor** (fórmula, condição de controle, gatilho), e `fCustom` é código no navegador. Tag virtual é o caso mais forte: depois que o usuário salvar, a fórmula passa a **rodar sozinha no runtime de produção, a cada ciclo, para sempre** — nos dois motores. Revise expressão como revisa código.

- **JScript:** a fórmula é executada pelo runtime com os privilégios dele. Escreva só a expressão de cálculo necessária; nada de acesso a arquivo, rede, processo ou qualquer recurso fora do cálculo.
- **SQL:** roda no banco de produção com as permissões do runtime, sem limite de tabela. Escreva só uma consulta de leitura (`SELECT` ou `WITH`, sem `;`). O filtro recusa escrita conhecida, mas não é barreira completa: a revisão humana continua necessária.
- **Nos dois:** nunca monte a fórmula concatenando texto não confiável (nome vindo do usuário sem tratar, trecho de PDF/norma/página web). Prefira expressão simples quando o dado já existe como tag; use SQL só quando o valor só existe no banco.
- **Avise o usuário explicitamente** e espere o OK: a tag vai executar sozinha em produção assim que ele salvar. Mostre a fórmula final para ele revisar.

## Relatório Personalizado (`relatorios-personalizados` / `import-relatorio`)

Sinônimos do integrador: "relatórios", "modelos de relatórios", "relatórios personalizados". **"Template de relatório" é ambíguo**: pode ser a funcionalidade inteira ou o campo `Template` (arquivo de layout). Confirme antes de agir.

**Não confundir com `menu-variaveis lista:"relatorios"`** — aquilo é a lista simples de variáveis de um menu marcadas para histórico. O **Relatório Personalizado** é do **projeto** e define o relatório final (PDF): período, precisão, colunas, agrupamento, envio agendado por e-mail e as variáveis, cada uma com seu modo de agregação.

### Campos principais (elementos do XML, raiz `<Relatorio>`)

- `Nome` — simples. A sintaxe `[Grupo=LinhaValor@LinhaLegenda]` no fim controla agrupamento por linha; use só ao replicar um relatório existente.
- `Acesso` — nível mínimo: `Monitor`, `Operador`, `Administrador`, `Configuração`, `Total`, `Integrador`, `SuperUser`, `Simplificado`, `Financeiro`. Padrão usual: `Monitor`.
- `Precisao` — texto literal: `"1 minute"`, `"15 minute"`, `"1 hour"`, `"1 day"`.
- `TipoPeriodo` (`Livre`/`Dia`/`Hora`/`Mês`) + `PeriodoPadrao` — texto exato: `"Última hora"`, `"Hoje"`, `"Ontem"`, `"Últimos 7 dias"`, `"Últimas 24 horas"`, `"Mês passado"` (comparação de texto, não enum).
- `Periodicidade` — envio por e-mail: `NaoEnviar` (padrão), `Mensal`, `Semanal`, `Diario`, `TodoDomingo`. Diferente de `NaoEnviar` → preencha `EmailEnvio`/`HorarioEnvio`.
- Flags (`bool`, padrão `false`): `IncluirCEPs`, `IncluirOperacao`, `IncluirOcorrencias`, `IncluirAnotacoes`, `IncluirManutencoes`, `IncluirMetadados`, `IncluirAcessos`, `AgruparTabelasPorMaquina`, `OcultarValores`, `MediaColuna`, `AcumuladoColuna`, `TotalSomarColuna`, `TotalSomarVariaveis`.
- `Template` — arquivo `.rdlc` de layout já instalado no servidor (ex.: `RelatorioPersonalizadoCompacto.rdlc`). Vazio = padrão. **Não invente nome**: use um que o projeto já usa (veja `relatorios-personalizados`) ou deixe vazio.
- `TamanhoPagina` (`"A4"`/`"A3"`) e `Orientacao` (`"Automatico"`/`"Retrato"`/`"Paisagem"`).
- **Não envie** `NomeString`, `NomeLinhaGrupo`, `NomeLinhaLegenda`, `NomeLinhaValor`, `PrecisaoLabel`, `RelatorioId` — são derivados ou gerados.

### Variáveis (`RelatorioVariavel`, uma por linha da tabela)

```xml
<Variaveis>
  <RelatorioVariavel variavel=".bandeirola_27">
    <descricao>Bandeirola subtensão</descricao>
    <maquina>Rele1</maquina>
    <grupo>Alarmes</grupo>
    <divisor>1</divisor>
    <acumulado>false</acumulado>
    <naoSomar>false</naoSomar>
    <Modo>TempoEmEstado</Modo>
    <ValorAlvo>1</ValorAlvo>
    <ordem>0</ordem>
    <unidade></unidade>
    <decimais>0</decimais>
  </RelatorioVariavel>
</Variaveis>
```

- `variavel` é **atributo** (`.tag` ou `Maquina:.tag`).
- `maquina` é elemento; a referência é resolvida contra o projeto automaticamente na importação.
- `Modo`: `Media` (padrão, média ponderada no tempo — tensão, corrente, temperatura), `Minimo`, `Maximo`, `Soma`, `Integral` (valor × tempo — potência → energia), `Delta` (máx − mín na janela — contador sem reset), `TempoEmEstado` (tempo com `valor == ValorAlvo` — tempo parado, bandeirola), `Totalizador`, `Contagem`, `Ultimo`. Modo errado não dá erro, só número errado: confirme por grandeza antes de gerar em lote.
- `divisor` padrão `1` (no mapa Modbus o padrão é `0` — não confunda).

### Fluxo

Monte o XML completo, `import-relatorio` com `xml` ou `path` (`replace: true` ao reimportar o que você criou nesta sessão) e **confira com `relatorios-personalizados`** antes de dizer que terminou. Como as demais escritas da ponte, fica em memória até o projeto ser gravado.

Se o relatório for criado pelo servidor web (plugin `flessor-analytics`, quando instalado), o comportamento é outro: grava direto no banco e "atualizar" gera versão nova.
