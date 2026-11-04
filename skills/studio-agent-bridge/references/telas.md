# Telas: leitura, convenções visuais e controles avançados

## "Explique essa tela" — como ler o XML de `screen-xml`

Não existe comando que devolva a tela explicada em português — isso é trabalho seu. `screen-xml` (`menu`, `screen`) devolve o XML cru e completo; interprete os atributos. Um `Controle` por elemento:

| Atributo | Significado |
| --- | --- |
| `Variavel` | Tag/expressão exibida pelo controle (o valor ao vivo não vem da ponte). |
| `VariavelPiscar` (alias `VariavelComando`) | Expressão que faz o controle piscar — mesmo campo com dois nomes. |
| `<Invisivel>` (elemento, ver abaixo; alias `Corpo`) | **Nome enganoso:** a expressão torna o controle **invisível** quando verdadeira. Não inverta a leitura. |
| `Ativo` | Condição "ativo quando" — liga/desliga o estado do controle. |
| `Cor_Ativo` / `Cor_Inativo` (aliases `Cor_Titulo`/`Cor_Fundo` em `fCaixa`) | Cor por estado. **Semântica invertida** numa variável tipo "Falha": o vermelho fica em `Cor_Ativo`. Leia o nome da variável antes de concluir qual cor é a normal. |
| `Texto` / `TextoVerdadeiro`/`TextoFalso` / `Texto_Ativo`/`Texto_Inativo` | Rótulo fixo / rótulo por estado. |
| `Valor_Maximo`/`Valor_Minimo`/`Valor_Alarme`/`Valor_Trip` (+ `Variavel_Valor_*` quando o limite vem de outra tag) | Limites de gauge/gráfico/faixa. |
| `Maquina` no próprio controle | Pode ser **diferente** da máquina padrão da tela. Agrupe por esse atributo quando presente. |
| `Dica` | Tooltip — normalmente já é uma explicação em português; aproveite. |
| `Type` | Tipo do widget (`fLed`, `fCaixa`, `fBotao`, `fGraph`, `fTapButton`…) — decide quais atributos fazem sentido. `fCaixa` é contêiner de layout, sem dado próprio. |

Fora dos controles, a tela tem `Fundos`: `Variavel`/`Valor`/`UidFundo` — troca a imagem de fundo da tela inteira quando a condição bate (ex.: unifilar mudando conforme o estado da rede).

Para explicar a um humano: agrupe por proximidade/`ParentID` (uma `fCaixa` costuma ser uma "unidade" — comporta, gerador, equipamento), use `Texto`/`Dica` como o nome que o operador reconhece e `Variavel`/`Cor_Ativo`/`Invisivel` como "o que mostra e quando muda". Não invente significado de tag só pelo nome.

### Formato do "invisível quando"

É o **elemento** `<Invisivel>`, não atributo: `<Controle ...><Invisivel>!.tag</Invisivel></Controle>`. Fora do `fCustom`, os atributos `Visivel="..."` e `Invisivel="..."` são descartados em silêncio. **Não copie o `<Corpo />` vazio** das telas exportadas para um controle que tem `<Invisivel>`: os dois gravam no mesmo campo e o vazio zera a expressão.

### Arquivo de tela avulsa (`.screen.flx`)

Usado em "Importar tela" do menu. Raiz `<Tela Nome=... Descricao=... Nivel="Monitor">` com `<Controles>`, `<Fundos />`, `<NomeMenu>`, `<TagsDeEntrada />`, `<Width>` e `<Height>` (resolução; 3840×2160 para 4K). O Studio exporta como `Menu.Tela.screen.flx`. Vale a posição relativa `Posi_X/Y`; a absoluta é recalculada pelo `ParentID`. Cores aceitam ARGB inteiro ou `#AARRGGBB` (o Studio regrava em hex).

## Padrões de referência

Ao montar tela, mapa ou alarme novo, prefira (dentro do que o Studio permite):

- **ISA-101** (HMI de alto desempenho): fundo neutro no estado normal; cor saturada (vermelho, amarelo, laranja) **só para condição anormal**. Hierarquia: visão geral → área → equipamento → diagnóstico. Sem 3D/gradiente decorativo.
- **ISA-18.2** (alarmes): cada ocorrência exige ação do operador; prioridade consistente; cuidado com chattering (falta de histerese).
- **ISA-112** (SCADA): pano de fundo de arquitetura e ciclo de vida.
- **IEC 63303:2024**: se a decisão precisar bater com cláusula específica, sinalize a incerteza e confirme com o usuário — não invente o que a norma diz.
- **SI**: unidade SI nos rótulos (V, A, Hz, °C, kPa, W, m, s, kg). Mantenha a unidade do fabricante/projeto quando já for convenção (RPM, bar), sempre explícita no rótulo.

Se o projeto já usa verde/vermelho para estados normais (aberta/fechada) e o usuário pedir consistência com uma tela existente, priorize a consistência. Sem esse pedido, siga ISA-101.

## Estrutura de tela de controle/status por equipamento

Cada unidade vira uma **caixa (`fCaixa`)** com seções empilhadas, cada uma com cabeçalho em negrito:

1. **Status** — LED (`fLed`, 12×12 px) + rótulo (`fFixo`) colado à direita, por indicador booleano.
2. **Dados** (opcional) — pares rótulo + valor (`fLabel` ligado à `Variavel`).
3. **Comando** (só se o equipamento aceita controle) — botões `fTapButton` lado a lado ("Abrir"/"Fechar"), sempre com `Confirmar="true"` e `Nivel="Operador"`.

Unidades iguais (comporta 1/2/3) = mesma caixa repetida lado a lado com espaçamento fixo (ex.: 270 px por caixa).

## Posição dentro de uma caixa — `Posi_X`/`Posi_Y` é o que manda

Todo controle tem `Posi_Absoluta_X/Y` (no canvas) e `Posi_X/Y` (relativo ao `ParentID`). **O relativo governa o layout dentro da caixa.** Fórmula, sempre:

```
Posi_X do filho = Posi_Absoluta_X do filho − Posi_Absoluta_X da caixa (ParentID)
Posi_Y do filho = Posi_Absoluta_Y do filho − Posi_Absoluta_Y da caixa (ParentID)
```

Erro típico: usar um deslocamento aproximado copiado de outra caixa. Funciona com a caixa no topo e quebra numa caixa mais abaixo — o filho cai fora da área visível. Subtraia de verdade, controle por controle, usando o valor exato que você escreveu na caixa.

## Cores (ARGB inteiro)

- Verde `-13447886` · Vermelho `-65536` · Dourado `-10496` · Branco/neutro `-1`

Seguindo ISA-101: vermelho/laranja só para anormal de verdade (alarme, falha, fora de faixa). Status binário sem anomalia (comporta que opera nos dois estados) → tons neutros diferenciados por texto/posição; cor saturada só no LED de falha (`.CommunicationFault`, trip).

## Fontes

- Título da caixa (`fCaixa.Texto`): `Tahoma; 9,75pt`
- Cabeçalho de seção (`fFixo`): `Tahoma; 8pt; style=Bold`
- Rótulos e valores: `Tahoma; 8pt`
- Botões: `Tahoma; 9pt`

## Organização de menus e nomes

- **Menu novo dedicado** quando o equipamento/sistema é conceitualmente distinto; **menu temático existente** quando é mais um da mesma categoria (ex.: mais um switch no menu "Rede").
- As ocorrências do menu valem para todas as máquinas dele — a de "falha de comunicação" pode ser genérica.
- **Nome de máquina: curto, uma palavra, PascalCase, sem espaço** (`FW100`, `SW1`, `Diesel1`). Equipamento comercial: só o modelo (`PCC1301`), com marca/modelo/função completos na `dica`.
- Menus e telas podem ter espaço e nome descritivo ("Comportas", "Vertedouro").
- **Cada projeto é diferente.** Nunca reaproveite nome de tag/menu de outro projeto sem confirmar no atual. O que se reaproveita é o padrão visual.

## Controles avançados: `fCustom` e `fSVG`

Trate a primeira tela gerada com esses controles como protótipo: depois do `import-screen`, abra a tela renderizada (ou peça ao usuário) antes de considerar pronto — principalmente a parte de tempo real.

### `fCustom` — HTML/JS/CSS livre

- O HTML vai no atributo **`Visivel`** do XML. "Corpo" é só o nome exibido no editor; `Corpo="..."` no XML é ignorado.
- `ID` é manual e único: vira o `id` do elemento no navegador. ID repetido quebra em silêncio.
- **Placeholders `{chave}` e os fixos (`{maquina}`, `{id}`, `{top}`, `{left}`, `{width}`, `{height}`, `{ativo}`, `{lat}`, `{lon}`) resolvem UMA VEZ, ao carregar a página.** Para valor ao vivo, o seu próprio JS precisa ler o dado, por um destes caminhos:
  - elemento com `data-variavel="Maquina:.tag"` — a plataforma grava o valor em `data-value` a cada ciclo; atualizar o texto/estilo é com o seu script;
  - variável global `valoresVariaveis["Maquina:.tag"]` (atualizada a cada ~333–500 ms);
  - função global `avaliarExpressao("Maquina:.tag")`.
- `Variavel` ("Tags de Entrada") é um dicionário JSON `{"chave":"expressao"}` serializado como string no atributo.
- **Nunca escreva `:.` em lugar nenhum do HTML/CSS/JS** — a sequência é interpretada como referência de tag. Escreva decimal CSS sempre com zero (`0.5em`, não `.5em`). Um `:.` que passou no XML vira `$.` quando alguém abrir e salvar a tela no editor.
- **Sem sandbox.** O conteúdo roda como HTML/JS confiável no navegador de quem abrir a tela, com acesso à sessão logada. Nunca gere a partir de texto não confiável e avise o usuário que aquele código vai rodar no navegador de cada operador.

### `fSVG` — SVG animado por lista de variáveis (gráfico de barra, gauge, ponteiro)

O binding do `fSVG` **é ao vivo** (~333–500 ms): cada valor do dicionário passa pela engine de expressão completa (aritmética, referência entre máquinas).

- O SVG vai no atributo **`Texto`** do XML (o editor mostra como `SvgSource`). Aceita SVG inline, URL ou arquivo. Prefira **inline** — URL/arquivo indisponível vira SVG vazio sem aviso.
- Chave do dicionário: `"idDoElemento__Propriedade"` (2 partes) ou `"id__Transform__rotate"` (3 partes, só rotação). N variáveis num SVG = N entradas no dicionário, sem N controles.
- Todo elemento animado precisa de `id=` definido por você. O dicionário referencia o **id original** (o prefixo que aparece no navegador é interno).
- Valor aceita literal, `Maquina:.tag`, aritmética (`(.nivel-20)*1.8-90`), `var(--nome)`/`--nome`.
- Propriedades: nomes em PascalCase viram kebab-case (`StrokeWidth`→`stroke-width`, `FontSize`, `FillOpacity`); `CenterX`/`CenterY`/`RadiusX`/`RadiusY` → `cx`/`cy`/`rx`/`ry`. As mais seguras: `Fill`, `Stroke`, `Opacity`, `Width`, `Height`, `X`, `Y`, `X1`/`Y1`/`X2`/`Y2`. Propriedade fora disso: teste antes de confiar.

**Rotação (ponteiro) — siga à risca:**

1. O elemento já tem que ter `transform="rotate(0 cx cy)"` no SVG (pivô = centro do mostrador). Sem isso a agulha nunca gira, sem erro.
2. Chave: `"idDaAgulha__Transform__rotate"`.
3. Valor: ângulo em graus já mapeado (0–100 → −90°…+90°: `(.nivel/100)*180-90`).
4. Suavidade: `style="transition: transform .3s"` no elemento.

### Criação pela ponte

`import-screen` lê o XML direto; não precisa desenhar no editor. `Guid` do controle é gerado se vier vazio; `ID` é obrigatório e único. Parta de `screen-xml` de uma tela real e respeite a regra de `Posi_X/Y` relativo. Monte as duas metades juntas: o SVG estático (um `id` por elemento animado) e o dicionário `Variavel` (uma entrada por elemento+propriedade).

**Modelo — barras (uma por variável).** SVG cresce para baixo; para a barra subir, `Height` e `Y` vêm da mesma expressão:

```xml
<Controle Type="fSVG" ID="graf1" Posi_Absoluta_X="10" Posi_Absoluta_Y="10" Size_X="200" Size_Y="150"
  Maquina="Central"
  Texto="&lt;svg viewBox='0 0 200 150'&gt;
    &lt;rect id='barra1' x='10'  width='30' y='150' height='0' fill='#4a90d9'/&gt;
    &lt;rect id='barra2' x='60'  width='30' y='150' height='0' fill='#4a90d9'/&gt;
    &lt;rect id='barra3' x='110' width='30' y='150' height='0' fill='#4a90d9'/&gt;
  &lt;/svg&gt;"
  Variavel="{&quot;barra1__Height&quot;:&quot;.unidade1_Potencia*1.2&quot;,&quot;barra1__Y&quot;:&quot;150-(.unidade1_Potencia*1.2)&quot;,
             &quot;barra2__Height&quot;:&quot;.unidade2_Potencia*1.2&quot;,&quot;barra2__Y&quot;:&quot;150-(.unidade2_Potencia*1.2)&quot;,
             &quot;barra3__Height&quot;:&quot;.unidade3_Potencia*1.2&quot;,&quot;barra3__Y&quot;:&quot;150-(.unidade3_Potencia*1.2)&quot;}" />
```

(`150` = base do viewBox; `1.2` = px por unidade — ajuste ao range real.)

**Modelo — gauge com ponteiro:**

```xml
<Controle Type="fSVG" ID="gauge1" Posi_Absoluta_X="10" Posi_Absoluta_Y="10" Size_X="120" Size_Y="120"
  Maquina="Central"
  Texto="&lt;svg viewBox='0 0 120 120'&gt;
    &lt;circle cx='60' cy='60' r='55' fill='none' stroke='#ccc' stroke-width='6'/&gt;
    &lt;line id='agulha' x1='60' y1='60' x2='60' y2='15' stroke='#c0392b' stroke-width='3'
          transform='rotate(0 60 60)' style='transition: transform .3s'/&gt;
  &lt;/svg&gt;"
  Variavel="{&quot;agulha__Transform__rotate&quot;:&quot;(.nivelReservatorio/100)*180-90&quot;}" />
```

## Diagrama unifilar

**A linha para no contato, não passa por baixo do símbolo.** Um `fCondutor` contínuo com `fChave`/`fSimbolo` em cima aparece atravessando o disjuntor ou o trafo. Quebre o condutor em dois trechos que terminam nos contatos:

- `fChave` vertical: contatos em (x=32, y=6) e (x=32, y=h−7);
- TP com `Valor_Alvo=0`: dois círculos lado a lado, de x=1 até h/2+h−3, linha no meio da altura;
- TP com `Valor_Alvo=±90`: os mesmos círculos empilhados;
- Gerador, Terra e Para-raio: a linha termina na borda.

Tamanhos fixos: `fCondutor` espessura 6, comprimento mínimo 18. `fChave` 36×66 (66×36 com `Valor_Alvo=90`), contato perto de x=32. `fSimbolo` tem degraus de tamanho: Gerador 54/72/90; Transformador de Potencial 48×30, 66×42 ou 84×54; Terra e TC 30/42/54. "Caixa Tracejada" delimita área. Ao converter de outra referência: disjunção (quadrado) vira `fChave`; trafo vira `fSimbolo` "Transformador de Potencial" em linha com o ramal. Símbolo menor que o original é aceitável.

## Exportação para Elipse E3 (CSV)

Feita pelo Studio: menu de contexto da tela → **"Exportar tela para CSV (Elipse)"**. Para o resultado sair bom:

- **No E3 o primeiro objeto da lista pinta na frente** (inverso do Studio e do SVG). O Studio já grava na ordem certa; não reordene à mão.
- `fCondutor` e `fSVG` saem como desenho nativo do E3. Para um `fSVG` virar desenho nativo e animar:
  - use `rect`, linha reta, `circle` e `text`, com cor sólida em `fill`/`stroke`, sem `class`, `opacity` nem gradiente;
  - anime só `Height`, `Y` e `Text`;
  - ponha o que precisa ficar como figura (polígono, `path`, hachura) **antes** das formas simples no SVG — forma que vem antes de algo de figura vai junto para a figura.
- **O import do E3 lê um registro por linha física.** Quebra de linha dentro de campo trunca a tela dali em diante. O Studio grava scripts VB numa linha só e troca quebras de linha por espaço. Ao conferir o CSV: linhas físicas = 1 (cabeçalho) + número de registros.
- O CSV sai em Windows-1252: caractere fora dele (ex.: `Δ`) vira `?`. Desenhe esse caractere como figura.
