# Checkpoint 5 — Bug Hunt PetFiap

## Identificação

**Grupo:** 37

| Integrante | RM | Turma |
|---|---|---|
| Giovanna Fernandes Pereira | 565434 | 2CCPO |
| João Pedro de Moura Albino | 565323 | 2CCPO |
| Kauê Silva Matheus | 561675 | 2CCPO |

| Campo | |
|---|---|
| **Total de bugs corrigidos** | 12 / 12 |
| **Total de ajustes de Clean Code** | 6 / 6 |
| **Total de testes novos escritos** | 6 / 6 |
| **Suíte final (Run As → JUnit Test)** | **26 testes, 0 falhas** |

---

## Parte 1 — Bugs encontrados

| # | Sintoma observado (o que fiz/vi) | Causa raiz (arquivo e linha aproximada) | Correção aplicada | Conceito da disciplina |
|---|---|---|---|---|
| bug01 | Ao construir um atendimento, o nome do pet chegava como `null`, mesmo tendo sido informado ao Builder. | `AtendimentoBuilder.java`, método `comPet()`. O código fazia `petNome = petNome`, atribuindo o parâmetro a ele mesmo. | Alterei para `this.petNome = petNome`, armazenando o valor no atributo do Builder. | Builder, atributos de instância e uso de `this`. |
| bug02 | O Builder permitia tentar construir um atendimento sem dados obrigatórios, permitindo que um objeto inválido avançasse no fluxo. | `AtendimentoBuilder.java`, método `construir()`. Não existiam validações dos campos obrigatórios antes da criação. | Adicionei validações antes de chamar a Factory, lançando `IllegalArgumentException` quando os dados obrigatórios não são informados. | Builder, validação e invariantes de objeto. |
| bug03 | Ao solicitar um atendimento do tipo `TOSA`, a Factory criava o tipo concreto incorreto. | `AtendimentoFactory.java`, `switch` do método `criar()`. O caso `TOSA` instanciava a classe errada. | Corrigi o caso `TOSA` para retornar `new Tosa(...)`. | Factory Method, polimorfismo e criação de objetos. |
| bug04 | A consulta veterinária era criada, mas seus dados herdados, como pet, tutor, protocolo e data, não eram inicializados corretamente. | `ConsultaVeterinaria.java`, construtor. Os dados não eram repassados corretamente ao construtor da superclasse. | Passei os parâmetros para `super(protocolo, petNome, petPorte, tutorNome, dataHora)`. | Herança e chamada de construtor com `super`. |
| bug05 | Chamadas ao Singleton podiam produzir instâncias diferentes, comprometendo a sequência global dos protocolos. | `GeradorProtocolo.java`, método `getInstancia()`. A instância criada não era mantida corretamente no atributo estático. | Passei a armazenar a instância em `GeradorProtocolo.instancia` e retornar sempre a mesma referência. | Padrão Singleton e estado compartilhado. |
| bug06 | A verificação de conflito não reconhecia corretamente o mesmo nome de pet quando os objetos `String` tinham referências diferentes. | `AgendaService.java`, verificação realizada em `agendar()`. Era utilizada comparação com `==`. | Troquei a comparação por `.equals()`, comparando o conteúdo do nome do pet. | Igualdade de objetos, `==` vs `.equals()`. |
| bug07 | Mesmo para o mesmo instante, a verificação podia não reconhecer dois `LocalDateTime` equivalentes como o mesmo horário. | `AgendaService.java`, verificação de conflito em `agendar()`. O horário também era comparado por referência com `==`. | Substituí a comparação por `a.getDataHora().equals(novo.getDataHora())`. | Igualdade por valor em objetos e `LocalDateTime`. |
| bug08 | Quando um atendimento não existia, o service podia retornar `null` em vez da exceção de domínio esperada. | `AgendaService.java`, método `buscarPorId()`. Uma captura genérica de exceção impedia a propagação correta do erro. | Passei a utilizar `orElseThrow()` com `AtendimentoNaoEncontradoException`. | Exceções, `Optional` e fail-fast. |
| bug09 | A duração da Tosa continuava sendo 30 minutos em vez dos 60 minutos definidos no contrato. | `Tosa.java`, método de duração. O método tinha uma assinatura diferente e estava fazendo overload em vez de override. | Corrigi a assinatura para `getDuracaoMinutos()` e adicionei `@Override`, retornando 60. | Herança, sobrescrita e sobrecarga. |
| bug10 | O preço do Banho para porte `PEQUENO` não correspondia ao contrato do PetFiap. | `Banho.java`, método `calcularPreco()`. O valor da condição para porte pequeno estava incorreto. | Corrigi o retorno do porte `PEQUENO` para `60.0`. | Polimorfismo e regras de negócio. |
| bug11 | Era possível cancelar um atendimento em um status que não permitia cancelamento. | `Atendimento.java`, método `cancelar()`. O método alterava o status sem validar corretamente o estado atual. | Adicionei a validação para permitir a transição somente de `AGENDADO` para `CANCELADO`, lançando `StatusInvalidoException` nos demais casos. | Encapsulamento, máquina de estados e exceções de domínio. |
| bug12 | O sistema aceitava agendamentos com data e hora no passado e ainda poderia consultar o repositório antes de rejeitar a operação. | `AgendaService.java`, início do método `agendar()`. Não havia validação temporal antes do acesso ao repository. | Adicionei a validação com `LocalDateTime.now()` antes de qualquer consulta ao repositório, lançando `IllegalArgumentException` para datas passadas. | Validação de regra de negócio, fail-fast e separação de responsabilidades. |

## Parte 2 — Ajustes de Clean Code

| # | Onde estava | Qual princípio/boas práticas era violado | O que eu mudei |
|---|---|---|---|
| clean01 | `AtendimentoController.java` | Havia código morto para uma funcionalidade futura de fidelidade que não era utilizada pelo sistema, contrariando YAGNI e prejudicando a clareza. | Removi o método `calcularDescontoFidelidade()` e os comentários da funcionalidade ainda não implementada. |
| clean02 | `AgendaService.java` | O repository era injetado diretamente no campo com `@Autowired`, deixando a dependência menos explícita. | Troquei por injeção via construtor e deixei o atributo `repository` como `final`. |
| clean03 | `AtendimentoFactory.java` | O método `criar()` utilizava parâmetros de uma única letra (`p`, `t`, `n`, `po`, `tu`, `d`), dificultando a leitura. | Renomeei para `protocolo`, `tipo`, `petNome`, `petPorte`, `tutorNome` e `dataHora`. |
| clean04 | `GeradorProtocolo.java` | Existia um `System.out.println` de depuração dentro do construtor da classe de domínio. | Removi a saída `"GeradorProtocolo criado!"`, que não fazia parte da responsabilidade da classe. |
| clean05 | `AgendaService.java` | O service imprimia um recibo diretamente com `System.out.println`, misturando regra de negócio com saída de console. | Removi a impressão do recibo e mantive o método responsável apenas pelo fluxo de agendamento. |
| clean06 | `AgendaService.java`, método `agendar()` | O método concentrava validação de conflito, iteração e persistência, além de utilizar nomes pouco expressivos como `doPet` e `a`. | Extraí a verificação para `validarHorarioDisponivel()`, usando os nomes `atendimentosDoPet` e `atendimento`, deixando `agendar()` menor e mais legível. |

## Parte 3 — Testes novos (regras que estavam sem cobertura)

| # | Teste escrito (classe.método) | Regra coberta | Resultado ao escrever (vermelho/verde) |
|---|---|---|---|
| teste01 | `TosaTest.deveDurar60Minutos()` | A Tosa deve possuir duração de 60 minutos. | **Vermelho.** Revelou o bug de override da duração da Tosa. |
| teste02 | `BanhoTest.deveCustar60ReaisParaPortePequeno()` | O Banho de pet `PEQUENO` deve custar R$ 60,00. | **Vermelho.** Revelou o preço incorreto do Banho para porte pequeno. |
| teste03 | `ConsultaVeterinariaTest.deveCustar150Reais()` | A Consulta Veterinária possui preço fixo de R$ 150,00, independentemente do porte. | **Verde de cara.** A regra já estava implementada corretamente. |
| teste04 | `AgendaServiceTest.deveCancelarAtendimentoAgendado()` | Um atendimento `AGENDADO` pode ser cancelado e deve passar para `CANCELADO`. | **Verde de cara.** O caminho válido de cancelamento já funcionava. |
| teste05 | `AgendaServiceTest.deveRecusarCancelamentoDeAtendimentoJaCancelado()` | Um atendimento que já está `CANCELADO` não pode ser cancelado novamente. | **Vermelho.** Revelou que o método `cancelar()` não validava corretamente o status. |
| teste06 | `AgendaServiceTest.deveRecusarAgendamentoNoPassado()` | Um atendimento não pode ser agendado no passado e o repository não deve ser consultado nessa situação. | **Vermelho.** Revelou a ausência da validação de data antes do acesso ao repository. |

---

## Parte 4 — Perguntas de reflexão

### 1. A suíte como contrato (Aula 15)

A suíte de testes funcionou como um contrato executável do comportamento esperado do PetFiap. O projeto começou com 20 testes, sendo 9 vermelhos, e as mensagens de falha ajudaram a localizar os problemas. Por exemplo, quando um teste esperava o nome `Rex` e recebia `null`, foi possível seguir o fluxo até o `AtendimentoBuilder` e encontrar `petNome = petNome` em vez de `this.petNome = petNome`. Em vez de alterar várias partes do projeto ao mesmo tempo, corrigi uma causa e executei novamente a suíte para observar o efeito. Isso é mais seguro do que testar tudo manualmente com curl, porque os testes são repetíveis, rápidos e verificam automaticamente regressões. A cada correção ou refatoração, consegui executar novamente toda a suíte e confirmar que comportamentos anteriores continuavam funcionando.

### 2. Mock e injeção de dependência (Aulas 13 a 15)

No `AgendaServiceTest`, o Mockito cria um `AtendimentoRepository` falso através de `@Mock` e o `@InjectMocks` fornece esse objeto ao `AgendaService`. Em produção, essa responsabilidade é do container do Spring, que encontra o repository e injeta a dependência no service. Durante o Clean Code também alterei o `AgendaService` para receber o repository pelo construtor, deixando essa dependência explícita. No teste, não é necessário iniciar o Spring nem acessar um banco real, porque o comportamento do repository é programado com Mockito, por exemplo através de `when(...)`. Assim, o teste fica concentrado exclusivamente nas regras do `AgendaService`, além de executar mais rápido e de forma isolada.

### 3. `==` vs `.equals()` (Aula 7)

O problema da verificação de conflito estava no uso de `==` para comparar objetos. Em Java, `==` verifica se duas variáveis apontam para a mesma referência, enquanto `.equals()` verifica a igualdade de valor de objetos como `String` e `LocalDateTime`. Com Strings literais como `"Rex"`, `==` pode aparentemente funcionar por causa do String Pool e do internamento de literais, mas isso não garante que duas Strings de mesmo conteúdo sejam a mesma instância. O mesmo problema ocorria com objetos `LocalDateTime` diferentes representando exatamente a mesma data e hora. A correção passou a utilizar `.equals()` nas duas comparações. Assim, a regra de conflito passou a depender dos valores do pet e do horário, e não da identidade dos objetos na memória.

### 4. Sobrescrita vs sobrecarga (Aula 7)

O bug da Tosa mostrou uma diferença importante entre override e overload. A classe `Atendimento` já possuía `getDuracaoMinutos()`, mas a implementação da `Tosa` tinha uma assinatura diferente. Por isso, o Java tratava o método como uma sobrecarga, ou seja, um novo método, e não como substituição do comportamento herdado. Como consequência, quando o objeto era utilizado polimorficamente, continuava sendo chamada a implementação herdada que retornava 30 minutos. Corrigi a assinatura e utilizei `@Override`, fazendo a Tosa retornar os 60 minutos definidos no contrato. Se `@Override` estivesse presente desde o início, o compilador teria indicado que aquela assinatura não sobrescrevia nenhum método da superclasse, evitando o bug.

### 5. Singleton manual vs bean do Spring (Aula 14)

O `GeradorProtocolo` usa o padrão Singleton manual para garantir que exista uma única instância responsável pelo contador global de protocolos. O bug estava justamente na criação dessa instância: ela não era mantida corretamente no atributo estático, o que poderia reiniciar o estado e quebrar a sequência. A correção passou a armazenar e reutilizar a mesma instância em `getInstancia()`. O `AgendaService`, por outro lado, utiliza `@Service` e tem seu ciclo de vida administrado pelo Spring. Por padrão, um bean do Spring possui escopo singleton dentro do container, então a própria infraestrutura cria e reutiliza a instância. No `GeradorProtocolo`, nós mesmos somos responsáveis por implementar corretamente essa garantia; no `AgendaService`, essa responsabilidade pertence ao container do Spring.

### 6. Cobertura de testes: onde parar? (Aula 15)

Vale a pena manter também os testes que ficaram verdes logo ao serem escritos, porque eles documentam regras importantes e protegem o projeto contra regressões futuras. No PetFiap, nem todo teste novo precisava revelar um defeito naquele momento para ter valor. Alguns confirmaram que comportamentos previstos pelo contrato já estavam corretos, enquanto quatro testes revelaram bugs que ainda não eram cobertos pela suíte original. Em um projeto real com prazo, eu priorizaria primeiro as regras de negócio críticas, depois os principais caminhos de erro e casos de borda. O caminho feliz também precisa ser protegido, mas buscar 100% de cobertura apenas como número não garante qualidade. É mais importante que os testes cubram comportamentos relevantes e riscos reais do sistema.

---

## Parte 5 — Espaço livre (opcional)

A principal dificuldade do checkpoint foi perceber que nem todos os problemas estavam diretamente associados aos testes que já estavam vermelhos. Alguns comportamentos só ficaram evidentes ao criar os testes que faltavam e outros exigiram leitura cuidadosa do código. O exercício também mostrou a importância de fazer alterações pequenas e executar a suíte após cada correção ou refatoração. Ao final, a suíte ficou com 26 testes executados e 0 falhas.
