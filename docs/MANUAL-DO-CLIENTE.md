# Manual do cliente — WaSap for WHMCS

Este manual descreve a instalação e a operação do addon **WaSap for WHMCS**. Os nomes em negrito reproduzem a interface em português do módulo. Faça primeiro uma implantação de teste e nunca publique tokens, secrets, cookies, URLs com tokens ou respostas completas da API em chamados e capturas de tela.

## 1. Requisitos e preparação

Antes de começar, confirme:

- WHMCS 8.13.1 ou superior, com acesso administrativo e ao sistema de arquivos;
- somente PHP 8.2, 8.3 ou 8.4, com ionCube Loader 15;
- um plano [WaSap](https://wasap.com.br/) ativo, com pelo menos uma conexão de WhatsApp habilitada;
- para usar templates de mensagens com emojis, banco de dados compatível com UTF8MB4 e `$mysql_charset = 'utf8mb4'` na configuração do WHMCS;
- a opção **Tentativa de carregar todos os arquivos** habilitada para o carregamento dos arquivos codificados;
- PHP cURL habilitado para as chamadas HTTPS à API WaSap e ao endpoint `includes/api.php` do próprio WHMCS;
- cron normal do WHMCS configurado e executando regularmente;
- banco de dados e arquivos completos em backup;
- **System URL** e **SSL System URL** do WHMCS apontando para a URL pública canônica em HTTPS.

O entrypoint `modules/addons/wasap_for_whmcs/wasap_for_whmcs.php`, os carregadores de hooks `modules/addons/wasap_for_whmcs/hooks.php` e `includes/hooks/wasap_for_whmcs.php` e o executor de cron `modules/addons/wasap_for_whmcs/cron/process_queue.php` exigem ionCube Loader 15. O endpoint público `wasap-login.php` é PHP aberto, mas depende de uma instalação funcional do addon; portanto, isso não torna ionCube opcional.

Escolha o pacote pela versão `major.minor` exibida por `php -v`: instale
`wasap-for-whmcs-php-8.2.zip` para PHP 8.2,
`wasap-for-whmcs-php-8.3.zip` para PHP 8.3 ou
`wasap-for-whmcs-php-8.4.zip` para PHP 8.4. Web e cron devem usar o mesmo alvo;
não reutilize um ZIP em outra versão do PHP. Os três artefatos usam ionCube
Encoder 14.0.0 e Loader 15. Antes de copiar, abra
`release-metadata.json` e confirme módulo 5.0.4, alvo PHP e essas versões.

## 2. Instalação do addon

1. Extraia o pacote em uma máquina segura.
2. Localize a pasta `modules/addons/wasap_for_whmcs/` do pacote.
3. Envie **a pasta inteira**, preservando nomes, subpastas e permissões, para a instalação do WHMCS. O resultado deve ser:

   ```text
   <raiz-do-whmcs>/modules/addons/wasap_for_whmcs/
   ```

4. Não renomeie a pasta: o identificador esperado pelo WHMCS é `wasap_for_whmcs`.
5. Garanta que o usuário do PHP possa ler os arquivos e que o usuário/banco configurado no WHMCS possa criar e alterar as tabelas do módulo durante a ativação.
6. Não copie apenas o arquivo principal. O diretório `src/`, os catálogos `lang/` e os demais componentes do addon são necessários.

## 3. Instalação do endpoint `wasap-login.php`

1. Copie o arquivo `wasap-login.php` fornecido no pacote para a **raiz pública da mesma instalação do WHMCS**, ao lado de `init.php` e `clientarea.php`:

   ```text
   <raiz-do-whmcs>/wasap-login.php
   ```

2. Preserve o nome exato do arquivo e permita sua leitura pelo PHP.
3. Confira que a URL pública `https://clientes.exemplo.invalid/wasap-login.php` chega à mesma instalação. Um acesso sem token pode mostrar uma resposta de validação; isso é esperado e não é um teste de login bem-sucedido.
4. Não mova o endpoint para `modules/` e não o proteja com login prévio, Basic Auth ou regra que destrua/recrie a sessão. Ele precisa receber o token antes do roteamento autenticado da área do cliente.
5. Se houver proxy reverso, CDN, balanceador ou múltiplos nós, mantenha HTTPS, host, porta, cookies e sessões coerentes em todo o percurso. Todos os nós devem compartilhar o mesmo banco; as sessões PHP também devem ser compartilhadas ou manter afinidade durante o fluxo.

> **Segurança:** nunca use um token real na documentação ou em testes públicos. O link contém um token opaco e deve ser tratado como credencial temporária.

## 4. Ativação no painel administrativo

1. Entre no painel administrativo do WHMCS com uma função autorizada a gerenciar addons.
2. Abra **Configurações do Sistema > Módulos Addon** (em versões/temas antigos, **Setup > Addon Modules**).
3. Localize **WaSap for WHMCS** e clique em **Ativar**.
4. Defina o controle de acesso para os grupos/funções administrativas que poderão operar o addon.
5. Clique em **Configurar**, preencha os campos descritos na próxima seção e salve.
6. Acesse o módulo pelo menu **Addons > WaSap for WHMCS**. A ativação cria/atualiza as tabelas de fila, registros, templates, preferências e tokens.

Se a ativação falhar, não repita indefinidamente: confira o log PHP/WHMCS, permissões do banco, integridade do upload e versão do PHP. Corrija a causa e só então tente novamente.

## 5. Configuração de envio e da API WaSap

Em **Configurações do Sistema > Módulos Addon > WaSap for WHMCS > Configurar**, preencha os campos a seguir.

1. **Módulo Ativo** — marque para habilitar as notificações.
2. **Base URL da API** — informe somente a origem HTTPS autorizada fornecida pela WaSap, sem acrescentar manualmente `/api/messages/send`. Exemplo fictício: `https://suaidAPI.wasap.com.br`.
3. **Token WaSap** — cole o token fornecido pela WaSap. O cliente HTTP o envia como Bearer; não escreva `Bearer ` no campo, salvo orientação expressa do provedor. Nunca reutilize o exemplo deste manual nem exponha o valor salvo.
4. **WaSap User ID** — informe o `userId` (ID do atendente).
5. **WaSap Queue ID** — informe o `queueId` (ID do setor/fila).
6. **WaSap WhatsApp ID** — informe o `whatsappId` (ID da conexão WhatsApp), caso não seja informado, será utilizada a Conexão definida como Principal no sistema do WaSap.
7. **Disable Bot** e **Open Ticket** — escolha `true` ou `false` conforme o comportamento desejado na WaSap. 
8. **Números dos Administradores** — coloque um destinatário por linha, sempre com DDI e apenas dados válidos. Formatos aceitos:

   ```text
   5511900000000|12|portuguese-br - Número de telefone, ID do Admin (opcional) e idioma para envio das notificações administrativas.
   15550000000|7|english - Número de telefone, ID do Admin (opcional) e idioma para envio das notificações administrativas.
   34910000000|spanish - Número de telefone e idioma para envio das notificações administrativas.
   5511900000000 - Número de telefone, como não consta o idioma preferido, será enviado aquele que estiver definido nas configurações do WaSap ou do WHMCS.
   
   ```

   O primeiro campo é o telefone; ID administrativo e idioma são opcionais. Também é aceito `telefone|idioma`. Sem idioma, o módulo usa o idioma da conta administrativa quando houver ID, ou o padrão da instalação. Os números acima são deliberadamente fictícios.
9. **Consentimento** — marque **Entendo os riscos de automações no WhatsApp**.
10. **Envios por minuto (máximo 20)** — informe de `1` a `20` (padrão `5`). Na prática, o teto é aplicado por execução do processador; com o hook padrão, por ciclo do cron do WHMCS.
11. **Fila de Envio** — mantenha marcada para suavizar picos; desmarcada, os eventos compatíveis usam envio imediato.
12. Selecione o campo em **Optin**, conforme a seção [Preferências e consentimento dos clientes](#11-preferências-e-consentimento-dos-clientes), e salve.

Depois de salvar, abra o addon. O aviso **Atenção: revise as configurações** lista campos obrigatórios ausentes ou inválidos.

## 6. Credenciais WHMCS para `CreateSsoToken`

Para que o login único funcione de forma recomendada:

1. No WHMCS, abra **Configurações do Sistema > Credenciais da API**.
2. Crie uma credencial dedicada ao módulo, vinculada a uma função administrativa de privilégio mínimo.
3. Dê a essa função acesso à API e permissão para a ação `CreateSsoToken`.
4. Se o WHMCS restringir a API por IP, autorize o IP de saída do próprio servidor WHMCS (ou do nó que atenderá `wasap-login.php`).
5. Copie o identificador para **WHMCS API Identifier (SSO)** e o secret correspondente para **WHMCS API Secret (SSO)**.
6. Salve e proteja o secret como uma senha. Não o inclua em logs, exemplos, tickets ou backups sem criptografia.
7. Confirme que `https://seu-host/includes/api.php` pertence exatamente à instalação escolhida e é acessível pelo servidor via cURL/TLS.

Os dois campos devem representar o mesmo par. Quando ficam vazios, existe um caminho de compatibilidade via `localAPI`, mas a credencial dedicada e o teste do fluxo HTTP são preferíveis para diagnosticar e controlar permissões.

## 7. Aba **Templates**

1. Abra **Addons > WaSap for WHMCS > Templates**.
2. Em **Idioma**, escolha **Padrão do módulo** para editar o modelo base, ou escolha um idioma instalado/detectado no WHMCS; clique em **Carregar idioma**. Uma versão de idioma sobrescreve o modelo base para destinatários daquele idioma, enquanto a ausência de versão usa os fallbacks do módulo.
3. Localize o template nas categorias **Clientes**, **Faturas**, **Serviços**, **Domínios**, **Administradores**, **Tickets** ou **Gerais**. O indicador `(?)` explica quando o evento dispara.
4. Use o switch no cabeçalho para alternar individualmente entre **Ativo** e **Desativado**.
5. Para ações em lote, use **Ativar grupo** ou **Desativar grupo** na categoria, ou **Ativar todos** / **Desativar todos** no topo.
6. Clique em **Editar**, altere **Título** e **Mensagem**, e use **Inserir variável rápida...** em **Variáveis WHMCS**. As variáveis mantêm o formato `{$nome_da_variavel}`. Use somente as oferecidas para o contexto do template; por exemplo, `{$login_unico_wasap}` depende da configuração descrita abaixo.
7. Clique em **Salvar este template**. Salvar um idioma selecionado cria ou sobrescreve a versão correspondente, sem substituir silenciosamente todos os demais idiomas.
8. **Desfazer** restaura somente aquele template/idioma para o modelo original e preserva seu estado de ativação.
9. **Restaurar padrões** repõe em massa os templates e emojis originais. **Aplicar templates sem emojis** repõe em massa as versões sem emojis. Leia a confirmação: ambas podem sobrescrever personalizações dos modelos padrão; faça backup antes.
10. O template de nova venda também apresenta **Filtros desta mensagem de nova venda**: **Grupos de produtos**, **Produtos**, **Valor mínimo** e **Valor máximo**. IDs são separados por vírgula; campos vazios notificam todas as vendas elegíveis.

Para links únicos, marque **Variável {$login_unico_wasap}**. A opção **Token nas URLs das notificações** controla separadamente se URLs normais de faturas, tickets e outros avisos recebem o fluxo de login; desative-a se quiser conservar URLs comuns e usar login apenas no template explícito.

## 8. Abas **Teste**, **Registros** e **Diagnóstico**

### **Teste**

1. Informe **Número com DDI**, somente com os dígitos do país e do assinante.
2. Edite **Mensagem** sem inserir informações sensíveis.
3. Clique em **Enviar teste**.
4. Uma mensagem de sucesso confirma que a API aceitou o envio; ela não garante leitura ou entrega final pelo WhatsApp. Em caso de falha, guarde o horário e o erro sanitizado.

O teste usa **Base URL da API**, **Token WaSap**, os três IDs, **Disable Bot** e **Open Ticket** atuais. Se **Registros** estiver ativo, o resultado também aparece no histórico.

### **Registros**

- Os cartões mostram **Fila pendente**, **Enviadas**, **Falhas** e **Modo de envio** (**Fila** ou **Imediato**).
- A tabela exibe ID, **Pedido**, **Destinatário**, **Mensagem**, **Evento**, **Status**, **Criado em**, **Enviado em**, **Tentativas** e **Resposta**.
- **Processar fila agora** (no cabeçalho do painel) executa um ciclo manual e informa quantos itens foram processados.
- O botão **Registros: Ativo/Desativado** controla a retenção. Ao desativar, o histórico concluído é removido; pendências são preservadas. Novos itens terminais deixam de ser retidos.
- **Limpar registros** remove logs e itens `sent`/`failed`, mas preserva itens `pending`. Faça a coleta necessária antes de limpar.

Respostas podem conter dados operacionais. Restrinja o acesso administrativo e sanitize tudo antes de compartilhar.

### **Diagnóstico**

Use esta aba principalmente para SSO e carregamento dos hooks. Ela mostra:

- o **Diagnóstico de hosts** (host público, host de `includes/api.php` e último host de `CreateSsoToken`);
- o **Diagnóstico do fluxo de login**, incluindo último carregamento do hook global, `bootstrap`, `load_hooks.php`, `client_area_hooks.php`, callback `ClientAreaPage` e recepção de `wasap_login_token`;
- a última tentativa de `CreateSsoToken`, método, host, HTTP, resultado e categoria de erro;
- tokens em autenticação e tokens expirados ainda não normalizados.

Valores como **nunca registrado**, **nunca executado** ou **ainda não retornado** ajudam a localizar em qual etapa o fluxo parou. Eles não devem ser corrigidos editando diretamente o banco.

## 9. Fila e cron

Com **Fila de Envio** ativa, os hooks criam itens `pending`. O processador seleciona os itens vencidos em ordem de ID, respeita **Envios por minuto (máximo 20)** e chama a API. Sucesso gera `sent`; falha definitiva gera `failed`. Com **Registros** desativado, itens concluídos são removidos em vez de permanecerem no histórico.

O modo recomendado é o cron oficial do WHMCS: o addon registra `AfterCronJob` e processa uma vez ao final de cada ciclo. Portanto:

1. mantenha o comando de cron indicado pela sua própria versão do WHMCS;
2. confirme no WHMCS que ele conclui regularmente;
3. ajuste o limite considerando a frequência real: o rótulo diz “por minuto”, mas o limite técnico vale por ciclo;
4. use **Processar fila agora** somente para teste ou recuperação controlada, evitando execuções concorrentes.

O executor `modules/addons/wasap_for_whmcs/cron/process_queue.php` é uma alternativa manual do pacote e também exige ionCube Loader. Não crie um segundo agendamento direto além do cron oficial sem necessidade, pois dois processadores podem competir.

Um alerta de **Faturas pendentes antigas** indica itens `invoice_*` pendentes há mais de 30 minutos. O processador revalida o estado da fatura antes do envio (por exemplo, não envia uma fatura em rascunho ou com estado incompatível).

## 10. Links de login único

1. Ative **Variável {$login_unico_wasap}** e/ou **Token nas URLs das notificações**.
2. Em **Validade do link Wasap (minutos)**, selecione uma das opções fechadas: `5`, `15` ou `30` minutos; `1`, `2`, `3`, `4`, `6`, `12` ou `24` horas; ou `2`, `3`, `4`, `5`, `6` ou `7` dias. O padrão é **24 horas** (`1440` minutos).
3. Envie um template elegível a um cliente de teste e abra o link no navegador que conservará a sessão.

O link aponta para `wasap-login.php` e leva somente um token opaco. Cliente e destino permitido são recuperados do banco. Ao clicar em um link próprio ainda válido, o módulo solicita `CreateSsoToken`; o token SSO curto do WHMCS **não é criado na hora do envio**, e sua validade não é ampliada pelo campo do addon. O módulo aceita apenas a `redirect_url` segura devolvida pelo WHMCS quando esquema, host e porta coincidem.

Requisitos indispensáveis:

- **System URL**, **SSL System URL**, host do link e host externo devem identificar a mesma instalação e usar HTTPS;
- certificado TLS válido, DNS correto, relógio do servidor sincronizado e cURL disponível;
- cookies com domínio/path corretos e a mesma sessão lógica entre o clique, o WHMCS e o retorno;
- em cluster, banco compartilhado e sessão compartilhada ou afinidade consistente;
- nenhum cache deve armazenar a URL com token.

O link é individual, temporário e consumível. Não o encaminhe. Links vencidos, já consumidos, ligados a outro cliente/destino ou com correlação de retorno inválida são recusados. O fluxo pode aplicar bypass controlado do segundo fator somente ao cliente validado durante esse login único.

## 11. Preferências e consentimento dos clientes

Há duas camadas:

1. **Optin** obrigatório na configuração do addon: crie antes um campo personalizado de **cliente** no WHMCS (por exemplo, “Aceito receber notificações por WhatsApp”), disponibilize-o conforme sua política e selecione-o em **Optin**. Somente os valores exatos `Sim`, `Yes`, `Sí` e `1` (sem diferença entre maiúsculas/minúsculas e ignorando espaços nas extremidades) permitem notificações comuns. Opções com código e rótulo também são aceitas quando usam o formato delimitado `1 - Rótulo`, por exemplo `1 - Desejo receber avisos`. `Não`, `No`, `2`, `2 - Obrigado, talvez depois`, campo vazio, ausente ou qualquer valor desconhecido bloqueiam o envio. A validação é *fail-closed*: textos que apenas começam com `1`, sem o hífen delimitador e um rótulo, não autorizam notificações. Os aliases legados `S`, `Y`, `true` e `on` deixaram de ser aceitos; ajuste valores existentes para um dos formatos documentados antes de atualizar.
2. Preferências internas por categoria: faturas, tickets, serviços, domínios e segurança começam habilitadas; marketing começa desabilitado. A instalação atual mantém esses estados no banco para integração/customização, mas não apresenta na interface padrão um formulário geral para o cliente alternar todas as categorias.

Na área de perfil/segurança o cliente encontra a preferência de autenticação em duas etapas pelo WhatsApp e **Salvar preferência**. A ativação exige confirmação do código; ao ser ativada, mantém notificações de segurança habilitadas. O código essencial de autenticação em duas etapas é enviado por uma mensagem fixa, fora do catálogo de templates configuráveis, e não é bloqueado pelo opt-in comum nem pela preferência de categoria. O evento `security_login_token` também permanece essencial ao fluxo solicitado de autenticação.

Não marque opt-in em nome do cliente sem base legal. Registre consentimento, ofereça procedimento de revogação e, quando necessário, altere o campo personalizado para um valor não afirmativo. A revogação afeta novos eventos; itens já enfileirados devem ser revisados segundo sua política operacional.

## 12. Solução de problemas

### API ou teste falha

1. Confira **Módulo Ativo**, **Base URL da API**, **Token WaSap**, **WaSap User ID**, **WaSap Queue ID** e **WaSap WhatsApp ID**.
2. Não inclua o endpoint de envio nem `Bearer ` se a WaSap forneceu apenas a base/token.
3. Confirme DNS, firewall de saída, certificado e data/hora do servidor.
4. Faça um envio pela aba **Teste** e compare horário, HTTP e **Resposta** em **Registros**, ocultando credenciais.
5. Confirme na WaSap que conexão, atendente e fila estão ativos e relacionados entre si.

### cURL ou TLS

- Verifique `php -m` usando o **mesmo binário/SAPI** do cron e do servidor web; habilitar cURL apenas no CLI ou apenas no FPM não basta.
- Atualize o pacote de CAs e corrija a cadeia do certificado; não desative a verificação TLS como solução permanente.
- A categoria `cURL indisponível` ou `erro de conexão/TLS` na aba **Diagnóstico** separa esse caso de uma recusa HTTP da API WHMCS.

### Fila ou cron não anda

1. Veja o alerta **Fila sem processamento**; ele informa módulo desativado, token ausente ou tabela ausente.
2. Confirme a última execução bem-sucedida do cron oficial do WHMCS e o carregamento dos hooks.
3. Clique uma vez em **Processar fila agora**. Se funcionar, corrija o agendamento/SAPI; se não funcionar, revise a API e a estrutura do módulo.
4. Não interprete `Processed 0` como erro quando não há itens vencidos; `scheduled_at` futuro ainda não é elegível.
5. Se o executor direto reclamar de ionCube, use o cron normal/`AfterCronJob` nesta distribuição aberta ou instale a variante correta; não confunda esse stub com requisito geral do addon.

### SSO não autentica

1. Confira o endpoint na raiz e abra **Diagnóstico** após uma tentativa.
2. Se `CreateSsoToken` nunca executou, revise validade do link, carregamento de hooks e chegada a `wasap-login.php`.
3. Em `API recusou a solicitação`, confira identifier/secret, permissão `CreateSsoToken` e IP autorizado.
4. Em resposta inválida ou HTTP não 2xx, confira `includes/api.php`, logs do WHMCS e possíveis bloqueios de WAF.
5. Se há `redirect_url retornada`, mas não há sessão, compare HTTPS/host/porta, domínio/path/SameSite/Secure dos cookies, proxy headers, afinidade e armazenamento de sessões.
6. Teste em janela limpa, sem compartilhar o link. Um token consumido ou vencido deve ser recusado; gere outro em vez de editar a tabela.

### Sessão se perde ou entra no cliente errado

- Interrompa os testes e invalide o link. Verifique se todos os nós usam o mesmo banco, sessão e chave/configuração do WHMCS.
- Remova redirecionamentos entre `www`/sem `www`, HTTP/HTTPS ou portas diferentes no meio do fluxo.
- Não permita que CDN/proxy faça cache de `wasap-login.php` ou de respostas autenticadas.
- Confira se já havia outro cliente autenticado no navegador; repita em uma sessão limpa após corrigir a infraestrutura.

### Mensagem não é enviada

Verifique, nesta ordem: módulo e template ativos; opt-in afirmativo; preferência da categoria; telefone do cliente com DDI; evento realmente disparado; filtros de nova venda; item em **Registros**; `scheduled_at`; estado atual da fatura; IDs da conexão; resposta da API. Para avisos administrativos, confira também **Números dos Administradores** e o formato de cada linha. Um template desativado ou opt-out normalmente impede a criação do item, portanto pode não haver falha na fila.

## 13. Atualização e backup seguros

### Atualização entre versões com o identificador atual

1. Leia as notas da versão e confirme compatibilidade de WHMCS/PHP/ionCube.
2. Em janela de manutenção, suspenda temporariamente o cron e evite alterações de templates. Não desative/desinstale o addon apenas para atualizar.
3. Registre a quantidade de pendências e aguarde o escoamento quando possível.
4. Faça backup verificável:
   - arquivos atuais de `modules/addons/wasap_for_whmcs/` e `wasap-login.php`;
   - banco completo do WHMCS, incluindo todas as tabelas `mod_wasap_for_whmcs_*`, `tbladdonmodules` e configurações relacionadas;
   - templates personalizados, preferencialmente também em cópia offline segura.
5. Proteja o backup com criptografia e acesso restrito: ele pode conter mensagens, telefones, secrets e hashes/tokens. Não exporte esses dados para repositórios.
6. Extraia a nova versão fora da área pública, valide checksums/origem e compare os arquivos.
7. Substitua a pasta do addon de forma atômica quando possível; remova arquivos obsoletos da versão anterior sem apagar uploads ou customizações documentadas. Atualize também `wasap-login.php` com o arquivo da **mesma versão**.
8. Entre no painel e acesse/reative o módulo apenas se as notas exigirem, permitindo que o instalador aplique migrações. Não execute SQL manual não documentado.
9. Revise as configurações sem sobrescrever tokens mascarados, abra **Diagnóstico**, faça **Enviar teste**, processe uma pequena fila e teste um novo link único com cliente de homologação.
10. Reative o cron, acompanhe **Registros**, logs do WHMCS/PHP e pendências antigas.
11. Se houver regressão, interrompa cron/envios, restaure **arquivos e banco do mesmo ponto no tempo** e investigue antes de retomar. Restaurar apenas um dos dois pode deixar esquema, templates e código incompatíveis.

Após validar a atualização, mantenha o backup pelo prazo da política da empresa e descarte-o de modo seguro.

### Atualização/migração de uma instalação anterior

> **Contexto de migração:** o **identificador usado por versões anteriores** é reconhecido automaticamente. Não é necessário descobri-lo, informá-lo nem reproduzi-lo em uma instalação nova. Se uma migração manual excepcional exigir o nome exato, consulte a nota de versão fornecida separadamente do código-fonte.

1. **Faça backup antes de alterar arquivos ou dados.** Suspenda o cron e os envios, registre a fila pendente e crie uma cópia verificável e consistente:
   - da pasta atual completa da instalação do addon e de `wasap-login.php`;
   - da **pasta da instalação anterior**, se ela ainda existir, mantendo essa cópia fora da raiz pública;
   - do banco completo do WHMCS, incluindo as **tabelas criadas pela versão anterior**, as tabelas atuais, `tbladdonmodules` e configurações relacionadas;
   - dos templates e de qualquer customização documentada.
2. **Remova a pasta da instalação anterior do caminho ativo sem destruir o backup.** Preferencialmente, mova-a para uma área privada fora da raiz pública. Não mantenha as duas pastas de addon ativas ao mesmo tempo e não misture arquivos antigos e novos no mesmo diretório.
3. **Instale a pasta nova.** Copie integralmente a versão nova para `modules/addons/wasap_for_whmcs/`, preserve sua estrutura e permissões e substitua `wasap-login.php` pelo arquivo fornecido na mesma versão.
4. **Ative e migre.** No painel do WHMCS, localize **WaSap for WHMCS**, clique em **Ativar** ou acesse o módulo conforme as notas da versão para que o instalador reconheça os **registros legados do addon** e execute a migração. Não renomeie tabelas, não reproduza o identificador usado por versões anteriores e não execute SQL manual não documentado. Não desinstale o módulo anterior pelo painel se essa ação puder remover seus dados.
5. **Valide antes de retomar a operação.** Confirme configurações e templates, compare as quantidades de registros relevantes, abra **Diagnóstico**, faça **Enviar teste**, processe uma pequena fila e teste um link único novo com um cliente de homologação. Verifique também logs do WHMCS/PHP e se apenas os hooks da pasta nova são carregados; só então reative o cron.
6. **Faça rollback se a validação falhar.** Interrompa novamente cron e envios, retire a pasta nova do caminho ativo e restaure, do mesmo ponto no tempo, a pasta da instalação anterior, `wasap-login.php` e o banco completo. Não reverta apenas arquivos ou apenas tabelas. Confirme a operação da versão restaurada antes de liberar os envios e preserve os artefatos da tentativa para diagnóstico seguro.
