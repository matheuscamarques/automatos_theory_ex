# automatos_theory_ex
framework para teoria de automatos  em elixir


o código espelho que eu tentei algo assim anos atrás mas falhei mizeravelemente. Quero tentar mais uma vez.

```elixir 
defmodule MeuSistemaLogin do
  use MeuFramework.ModeloFSM # Importa as macros da DSL

  # Definições do Modelo FSM
  estados [:inicial, :aguardando_senha, :logado, :bloqueado, :erro_login]
  estado_inicial :inicial
  estados_finais [:logado] # Estados que representam um fluxo "bem-sucedido" para certos testes

  alfabeto_entrada [
    {:tentar_login, [:usuario, :senha]}, # Evento com parâmetros
    :senha_correta,
    :senha_incorreta,
    :tentativa_excedida,
    :logout,
    :timeout
  ]

  # Transições com guardas e saídas/ações esperadas
  transicao :inicial, {:tentar_login, %{usuario: u, senha: s}}, :aguardando_senha,
    guarda: &String.length(u) > 0 && String.length(s) > 0/1,
    saida_esperada: :prompt_senha_ok

  transicao :aguardando_senha, :senha_correta, :logado,
    saida_esperada: :login_sucesso

  transicao :aguardando_senha, :senha_incorreta, :inicial,
    saida_esperada: :login_falha,
    # Ação a ser verificada no oráculo (opcional)
    assercao_estado: &(&1.tentativas_login < 3)

  transicao :aguardando_senha, :tentativa_excedida, :bloqueado,
    saida_esperada: :conta_bloqueada

  transicao :logado, :logout, :inicial,
    saida_esperada: :logout_sucesso

  transicao :_, :timeout, :inicial # Transição de qualquer estado em timeout

  # Especificação da Estratégia de Teste
  # O usuário pode escolher uma ou mais estratégias
  gerar_testes com: :transition_tour

  gerar_testes com: :w_method,
    conjunto_caracterizacao: [:tentar_login, %{usuario: "test", senha: "pw"}],
      [:logout],
    max_estados_impl: 6 # Parâmetro para o W-method (m + n - 1)

  # Opcionalmente, definir como os parâmetros de entrada são gerados
  # para entradas parametrizadas como :tentar_login
  gerador_dados_entrada :tentar_login, fn ->
    %{usuario: Enum.random(["user1", "user2", ""]), senha: Enum.random(["pass", "", "wrong"])}
  end
end

``` 
