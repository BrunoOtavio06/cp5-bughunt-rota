# Checkpoint 5 — Bug Hunt PetFiap

## Identificação

**Grupo:** ROTA

| Integrante | RM | Turma |
|---|---|---|
| Letícia Gabrielle Andrade Temóteo | 563985 | 2CCPG |
| Bruno Otávio da Cruz Carvalho | 562354 | 2CCPG |
| Rafael Quattrer Dalla Costa | 562052 | 2CCPG |
| João Vitor Santana Silva Ribeiro | 564693 | 2CCPG |
| Rafael Louzã Lopes | 564963 | 2CCPG |

| Campo | |
|---|---|
| **Total de bugs corrigidos** | 12 / 12 |
| **Total de ajustes de Clean Code** | 6 / 6 (+2 extras: clean07 e clean08) |
| **Total de testes novos escritos** | 6 / 6 |
| **Suíte final (Run As → JUnit Test)** | 26 testes, 0 falhas |

---

## Parte 1 — Bugs encontrados

> As linhas indicadas na causa raiz referem-se ao código original recebido (commit `chore: estado original do projeto recebido`).

| # | Sintoma observado | Causa raiz (arquivo e linha aproximada) | Correção aplicada | Conceito da disciplina |
|---|---|---|---|---|
| bug01 | O Builder montava atendimento com nome do pet `null`. | `AtendimentoBuilder.comPet`: `petNome = petNome` alterava apenas o parâmetro. (`AtendimentoBuilder.java`, ~L24) | Alterado para `this.petNome = petNome`. | Encapsulamento / uso de `this` |
| bug02 | Builder aceitava atendimento sem nome ou sem porte. | `AtendimentoBuilder.construir` não validava campos obrigatórios. (`AtendimentoBuilder.java`, ~L39–43) | Adicionadas validações fail fast para nome e porte; removido o comentário que delegava essa validação ao controller. | Exceções / Fail Fast / Builder |
| bug03 | `TOSA` criava um `Banho`. | `AtendimentoFactory`: case `TOSA` instanciava `Banho`. (`AtendimentoFactory.java`, ~L17) | Factory passa a instanciar `Tosa`. | Factory / Polimorfismo |
| bug04 | Consulta era criada sem os dados recebidos. | Construtor de `ConsultaVeterinaria` chamava `super()` vazio. (`ConsultaVeterinaria.java`, ~L17) | Passagem dos argumentos para o construtor da superclasse. | Herança / construtores |
| bug05 | `getInstancia()` não mantinha uma única instância e a sequência reiniciava. | Singleton criava `new GeradorProtocolo()` sem guardar em `instancia`. (`GeradorProtocolo.java`, ~L17–21) | Criada instância única estática e `proximo()` sincronizado. | Singleton |
| bug06 | Conflito do mesmo pet e horário não era detectado. | `AgendaService.agendar` comparava `String` e `LocalDateTime` com `==`. (`AgendaService.java`, ~L23) | Troca por `.equals()`. | Igualdade de objetos |
| bug07 | Busca por ID inexistente retornava `null`. | `AgendaService.buscarPorId` capturava qualquer exceção e a engolia. (`AgendaService.java`, ~L37–42) | Removido `catch` genérico; `AtendimentoNaoEncontradoException` passa a propagar. | Exceções específicas |
| bug08 | Preço de banho pequeno/grande estava invertido. | `Banho.calcularPreco` retornava 100 para PEQUENO e 60 no fallback. (`Banho.java`, ~L26–33) | Corrigido para 60/80/100. | Polimorfismo / regra de negócio |
| bug09 | Tosa retornava duração herdada de 30 min. | Método `getDuracaoMinutos(String)` era sobrecarga, não sobrescrita. (`Tosa.java`, ~L40) | Alterado para `getDuracaoMinutos()` com `@Override`. | Override vs overload |
| bug10 | Era possível agendar atendimento no passado. | `AgendaService.agendar` não validava data/hora antes do repositório. (`AgendaService.java`, ~L20–21) | Adicionada validação com `IllegalArgumentException` antes de qualquer consulta. | Fail Fast / regra de negócio |
| bug11 | Atendimento concluído ou cancelado podia ser cancelado novamente. | `Atendimento.cancelar` mudava o status sem validar o estado atual. (`Atendimento.java`, ~L63–65) | Agora só `AGENDADO` pode virar `CANCELADO`; demais lançam `StatusInvalidoException`. | Máquina de estados / exceções |
| bug12 | Entidade nova podia chegar ao JPA sem estratégia de geração de ID. | Campo `id` tinha apenas `@Id`. (`Atendimento.java`, ~L14–15) | Adicionado `@GeneratedValue(strategy = GenerationType.IDENTITY)`. | JPA / persistência |

## Parte 2 — Ajustes de Clean Code

| # | Onde estava | Princípio/boa prática violado | O que mudou |
|---|---|---|---|
| clean01 | `AtendimentoFactory.criar` | Parâmetros de uma letra prejudicavam legibilidade. | Renomeados para `protocolo`, `tipo`, `petNome`, `petPorte`, `tutorNome`, `dataHora`. |
| clean02 | `AgendaService` | Field injection escondia dependência e dificultava testes. | Trocado `@Autowired` em campo por injeção via construtor. |
| clean03 | `AtendimentoController` | Mesmo problema de field injection. | Trocado por injeção via construtor. |
| clean04 | `GeradorProtocolo` | `System.out.println` de depuração dentro da classe de domínio. | Removida saída de console. |
| clean05 | `AgendaService.agendar` | Regra de negócio misturada com impressão de recibo no console. | Removido `System.out.println`; método apenas executa a regra. |
| clean06 | `AtendimentoController` | Código morto/YAGNI: método de fidelidade futura não utilizado. | Removido `calcularDescontoFidelidade`. |
| clean07 (extra) | `AtendimentoController.agendar` | Comentário enganoso/desatualizado: indicava só `tutorNome` como parâmetro e o exemplo usava uma data já no passado. | Comentário descreve todos os parâmetros e o exemplo usa data futura. |
| clean08 (extra) | `Atendimento` e `AgendaService.agendar` | Strings mágicas repetidas (`"AGENDADO"`, `"CONCLUIDO"`, `"CANCELADO"`). | Criadas as constantes `STATUS_AGENDADO`, `STATUS_CONCLUIDO` e `STATUS_CANCELADO`, usadas no lugar dos literais. |

## Parte 3 — Testes novos

| # | Teste escrito | Regra coberta | Resultado ao escrever |
|---|---|---|---|
| teste01 | `BanhoTest.deveCalcularPrecoDoBanhoConformePorte` | Banho custa 60/80/100 para pequeno/médio/grande. | Vermelho — revelou bug08. |
| teste02 | `TosaTest.deveDurar60Minutos` | Tosa dura 60 minutos. | Vermelho — revelou bug09. |
| teste03 | `ConsultaVeterinariaTest.deveCustar150ReaisIndependentementeDoPorte` | Consulta custa R$150 para qualquer porte. | Verde — regra já estava correta. |
| teste04 | `AgendaServiceTest.deveRecusarAgendamentoNoPassadoSemConsultarBanco` | Data/hora no passado deve falhar antes de acessar o banco. | Vermelho — revelou bug10. |
| teste05 | `AgendaServiceTest.deveCancelarAtendimentoAgendado` | Atendimento AGENDADO pode ser cancelado. | Verde — regra já estava correta. |
| teste06 | `AgendaServiceTest.deveRecusarCancelamentoQuandoAtendimentoNaoEstaAgendado` | CONCLUIDO e CANCELADO não podem ser cancelados. | Vermelho — revelou bug11. |

---

## Parte 4 — Perguntas de reflexão

### 1. A suíte como contrato (Aula 15)
Os testes foram usados como especificação executável. Quando um teste mostrava, por exemplo, que o nome esperado era `Rex` mas o valor retornado era `null`, o caminho foi seguir o dado desde o teste até o Builder e localizar a atribuição incorreta. O mesmo ocorreu com Factory, Singleton, serviço e regras de status. A vantagem sobre testar tudo manualmente com `curl` é a repetibilidade: a suíte verifica vários cenários em segundos e acusa regressões depois de cada mudança. Além disso, os testes unitários isolam a regra que falhou, sem depender de rede, Oracle ou da aplicação Spring completa.

### 2. Mock e injeção de dependência (Aulas 13 a 15)
Em produção, o Spring cria o `AtendimentoRepository` e o injeta no `AgendaService`. No teste, o Mockito assume esse papel: `@Mock` cria uma implementação falsa do repository e `@InjectMocks` monta o service usando essa dependência. Por isso o teste controla respostas como `findById` e `findByPetNome` sem acessar Oracle. A troca para injeção por construtor também deixa essa dependência explícita. Assim, produção e teste usam a mesma ideia de inversão de dependência, mas com responsáveis diferentes pela criação dos objetos.

### 3. `==` vs `.equals()` (Aula 7)
O conflito de horário usava `==` em `String` e `LocalDateTime`. Em Java, `==` compara se duas referências apontam para o mesmo objeto, enquanto `.equals()` compara o valor lógico. Literais como `"Rex"` podem parecer funcionar com `==` por causa do pool de Strings, mas isso é uma coincidência de implementação e não uma regra segura. No teste, outro objeto `LocalDateTime` com o mesmo horário fazia `==` retornar falso. A correção usa `.equals()` para comparar nome e data/hora pelo conteúdo.

### 4. Sobrescrita vs sobrecarga (Aula 7)
A classe `Tosa` tinha `getDuracaoMinutos(String porte)`, enquanto a superclasse define `getDuracaoMinutos()` sem parâmetro. Isso é overload: dois métodos com mesmo nome, mas assinaturas diferentes. Como o código chamava a versão sem parâmetro, era executado o método herdado de `Atendimento`, retornando 30 minutos. A correção foi declarar `getDuracaoMinutos()` e adicionar `@Override`. Com `@Override`, o compilador teria avisado imediatamente se a assinatura estivesse errada.

### 5. Singleton manual vs bean do Spring (Aula 14)
`GeradorProtocolo` precisa manter uma única instância porque o contador de protocolos é global. O bug estava em `getInstancia()`: quando a instância era nula, o método criava um novo objeto, mas não o armazenava, então cada chamada podia reiniciar o contador. A correção mantém uma instância estática única e sincroniza a geração do próximo número. Já `AgendaService` é anotado com `@Service`; por padrão, o próprio container Spring gerencia esse bean como singleton dentro do contexto da aplicação, sem precisar implementar manualmente `getInstancia()`.

### 6. Cobertura de testes: onde parar? (Aula 15)
Os testes que já nasceram verdes continuam úteis porque registram uma regra e impedem regressões. O teste do preço fixo da consulta, por exemplo, protege a regra de R$150 mesmo que ela já estivesse correta. Em um projeto real eu priorizaria primeiro regras críticas e caminhos de erro, depois o caminho feliz e casos de borda mais relevantes. Buscar 100% de cobertura apenas como número pode gerar testes sem valor. O objetivo é ter confiança sobre comportamentos importantes, especialmente aqueles que podem causar dados inválidos ou estados inconsistentes.

---

## Parte 5 — Espaço livre

Projeto corrigido preservando a estrutura original, sem alterar os 20 testes fornecidos. Foram adicionados somente os 6 testes exigidos pelo checkpoint.

Sobre o Clean Code: clean02 e clean03 aplicam o mesmo princípio (injeção por construtor) em classes diferentes, por isso registramos dois ajustes extras (clean07 e clean08). Além disso, o comentário enganoso do `AtendimentoBuilder` ("a validação dos campos obrigatórios fica por conta do controller") foi removido junto com a correção do bug02, já que a validação passou a ficar no `construir()`.
