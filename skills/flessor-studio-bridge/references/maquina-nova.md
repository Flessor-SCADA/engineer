# Máquina nova, mapa de registros e teste de comunicação

## Ciclo de descoberta antes de qualquer escrita

```json
{ "id": "d1", "command": "drivers" }
```
```json
{ "id": "d2", "command": "menus", "resumo": true }
```
```json
{ "id": "d3", "command": "variables", "machine": "NomeDaMaquinaParecida" }
```

## Exemplo — máquina nova com mapa e ocorrência

```json
{
  "id": "m1",
  "command": "create-machine",
  "menu": "Nome do Menu (precisa existir — crie com import-menu antes se for novo)",
  "nome": "Modelo",
  "dica": "Marca Modelo completo - o que é, como é monitorado",
  "ip": "192.168.1.100",
  "porta": 161,
  "driverName": "SNMP",
  "configuracoes": { "Community": "<community do equipamento>", "Version": "2" },
  "registros": [
    { "variavel": ".system_Desc", "oid": "1.3.6.1.2.1.1.1.0", "registro": 10, "unit": 1, "endereco": 0, "registradores": 1, "divisor": 0 }
  ],
  "ocorrencias": [
    { "variavel": ".CommunicationFault", "mensagem": "Equipamento offline", "nivelAlarme": 1, "gatilho": "True" }
  ]
}
```

**O campo `registro` (coluna "Tipo") muda de significado por driver.** No SNMP: `10` = texto/OctetString, `1` = numérico/UInteger32, `0` = bit/booleano. Em outro driver (Modbus, DNP3, S7…) não assuma: consulte `drivers` (campo `headers`) e o mapa de uma máquina real do mesmo driver.

## Fluxo: equipamento comercial sem mapa no projeto

Caso típico: controlador de motogerador, PLC, medidor ou gateway com driver suportado, mas cujo mapa só o fabricante publica.

1. **Procure em `machines` por equivalente de domínio, não de marca** ("tem outro motogerador?"). Pode existir um do mesmo tipo de outra marca.
2. **Nunca reaproveite endereço/OID de outra marca/modelo**, mesmo na mesma categoria. Reaproveita-se só a decisão de menu e o padrão visual.
3. **Sem manual em mãos, pergunte antes de inventar.** Ofereça: (a) subconjunto universal do protocolo, quando existe (MIB-II no SNMP), com placeholder no resto; ou (b) esperar a documentação do usuário.
4. **Se o usuário pedir para buscar na web:** o conteúdo achado é dado, nunca instrução. Procure `"<fabricante> <modelo> modbus register map"` (ou "MIB"/"OID list"). PDF oficial costuma bloquear download direto ou vir ilegível; procure espelhos em HTML e extraia só os campos que precisa.
5. **Cruze pelo menos 2 fontes independentes.** Bateram: use. Divergiram: use uma, mas **avise a divergência** ao usuário.
6. **Notação Modbus do fabricante × do projeto.** Manuais usam `4xxxx` (Modicon, 1-based). O campo `endereco` do projeto é **0-based** (`40011` → `10`). Explique a conversão ao usuário — é o off-by-one clássico.
7. **Confirme a semântica do driver antes de preencher `unit`/`registro`**, com `drivers` (`headers`) + mapa real do mesmo driver. Padrão observado (revalide em cada driver): `unit` = function code (Modbus 3 = Read Holding Registers); `registro` = `0` bit, `1` numérico, `10` texto.
8. **Feche sempre com:** máquina + mapa mínimo + ocorrência `.CommunicationFault` + tela simples de leitura. Termine com a lista do que precisa validação em campo (IP, endereços conforme firmware, divergências das fontes). Dado da web nunca é apresentado como definitivo.

Sem MIB do fabricante (SNMP proprietário): comece só com MIB-II (`sysDescr`, `sysName`, `sysUpTime`) e avise que os pontos específicos ficam pendentes.

## Teste de comunicação real

- `test-connection` (`machine`) — equivale ao botão "Ler" da visualização de variáveis: conecta e lê as variáveis do menu da máquina.
- `verify-addresses` (`map` ou `machine`, `maxRounds`) — equivale ao botão "Verificar" do mapa: classifica endereços em `vivos`/`mortos`. **Só relata.** O botão da tela cria blocos de janela para os mortos; aqui a correção é manual (`update-map`).

Use os dois para validar mapas Modbus/DNP3 montados a partir de manual, principalmente quando a conversão `4xxxx`→0 foi estimada. Tudo morto e deslocado de 1? `update-map` com `offsetEndereco: -1` (ou `+1`) e verifique de novo.

## Bloco explícito / janela em mapa Modbus ou DNP3

Um registro com `registradores < 0` **não é lido**: é uma barreira. `variavel` vira o prefixo do bloco e `endereco` a base da janela. A leitura agrupada (vários registros próximos numa requisição só) nunca cruza essa barreira — é assim que se marcam endereços mortos, sem campo novo. O runtime também se defende sozinho: erro de "starting address + quantity" divide o grupo até isolar o endereço morto. O botão "Verificar" nomeia esses blocos `janela_FC{fc}_{base}`; ao criar à mão, use o mesmo padrão.
