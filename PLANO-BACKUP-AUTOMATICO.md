# Plano de Backup Automático

## Objetivo

Criar cópias dos dados de amigos e festas fora do `localStorage`, evitando perda quando o usuário limpar o cache ou os dados do site.

## 1. Formato do backup

Reutilizar o formato JSON já usado pela exportação manual:

```json
{
  "versao": 1,
  "exportadoEm": "2026-09-29T12:00:00.000Z",
  "amigos": [],
  "festas": []
}
```

O arquivo deve registrar a data do backup e manter compatibilidade com futuras versões dos dados.

## 2. Geração automática

Executar o backup após alterações relevantes:

- cadastrar ou editar amigo;
- excluir amigo;
- cadastrar ou editar festa;
- excluir festa;
- importar dados.

O `localStorage` continuará sendo usado para acesso rápido e funcionamento offline, enquanto o backup será uma cópia externa.

## 3. Destino do backup

### Opção inicial: download automático

- gerar um arquivo JSON após as alterações;
- usar nomes como `meu-circulo-backup-2026-09-29-1430.json`;
- manter o `localStorage` como fallback caso o download não seja autorizado.

Limitação: muitos downloads podem ser incômodos e alguns navegadores podem bloqueá-los.

### Opção avançada: pasta escolhida pelo usuário

- solicitar a escolha de uma pasta uma única vez;
- usar a File System Access API para gravar o arquivo;
- atualizar `meu-circulo-backup.json` a cada backup;
- criar cópias históricas diárias ou semanais.

Essa alternativa depende do suporte do navegador, principalmente Chromium, e da permissão concedida pelo usuário.

## 4. Configurações da aplicação

Adicionar ao modal de configurações:

- ativar ou desativar backup automático;
- escolher a frequência: a cada alteração, diário ou semanal;
- escolher a pasta de backup;
- criar backup agora;
- restaurar backup;
- exibir data e hora do último backup.

## 5. Controle de frequência

Para evitar backups excessivos:

- executar no máximo um backup por intervalo configurado;
- criar backup somente quando os dados tiverem mudado;
- controlar alterações pendentes;
- manter, por exemplo, os sete backups mais recentes.

## 6. Restauração

Ao restaurar um arquivo:

1. selecionar o JSON;
2. validar `versao`, `amigos` e `festas`;
3. exibir um resumo dos dados encontrados;
4. pedir confirmação;
5. criar backup dos dados atuais;
6. gravar os dados restaurados no `localStorage`;
7. atualizar a interface.

## 7. Tratamento de falhas

Informar ao usuário quando:

- o navegador não suporta gravação automática;
- a permissão da pasta foi revogada;
- o arquivo não pôde ser gravado;
- o backup está atrasado;
- o último backup foi concluído com sucesso.

## Ordem recomendada de implementação

1. Backup automático por download após alterações.
2. Configuração de frequência.
3. Restauração com validação e confirmação.
4. Registro do último backup e tratamento de falhas.
5. Gravação em uma pasta escolhida pelo usuário.

## Observação importante

Nenhuma solução baseada exclusivamente no navegador garante proteção contra a limpeza dos dados do site. Para uma proteção mais robusta e sincronização entre dispositivos, a evolução recomendada é usar um banco de dados online, como Supabase, mantendo o backup local como camada adicional.
