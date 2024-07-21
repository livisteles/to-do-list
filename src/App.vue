<script setup>
import { computed, ref } from 'vue';
import { Axios } from "axios"

let pegarTarefas = ref('')

let tarefas = ref([])
let id = 1

const apiClient = new Axios({
  baseURL: "http://localhost:3000"
})

// async e await: 
// - async: é uma função assíncrona, roda sem atrapalhar o fluxo dos códigos
// - await: usado para aguardar as funções assíncronas
async function recuperarTarefas() {
  // resposta da requisição
  let response = await apiClient.get("/tarefas")

  tarefas.value = JSON.parse(response.data).map((tarefa) => {
    tarefa.id = parseInt(tarefa.id)

    return tarefa
  })

  for (let tarefa of tarefas.value) {
    if (id <= tarefa.id) {
      id = tarefa.id + 1
    }
  }
}

recuperarTarefas() // chamando a função que vai recuperar as tarefas


function addTarefa(event) {
  event.preventDefault()
  const novaTarefa = pegarTarefas.value.trim()

  if (novaTarefa) {
    const tarefa = {
      id: id,
      valor: novaTarefa,
      prioridade: 1,
      concluido: false,
    }

    const tarefaString = {
      ...tarefa,
      id: id.toString(),
    }
    // enviando a nova tarefa
    apiClient.post("/tarefas", JSON.stringify(tarefaString), {
      headers: {
        "Content-Type": "application/json"
      }
    })

    id++
    tarefas.value.push(tarefa)
    pegarTarefas.value = ''
  }
}

function excluirTarefa(id) {
  let tarefaIndex = tarefas.value.findIndex(tarefa => tarefa.id === id)
  if (tarefaIndex === -1) {
    return
  }

  let tarefa = tarefas.value[tarefaIndex]

  let confirmar = confirm(`Deseja realmente excluir a tarefa "${tarefa.valor}"?`)

  if (confirmar == true) {
    tarefas.value.splice(tarefaIndex, 1)

    // deletando a tarefa pelo id
    apiClient.delete(`/tarefas/${tarefa.id}`)
  }
}

function mudarPrioridade(id, prioridade) {
  let tarefa = tarefas.value.find(tarefa => tarefa.id === id)
  if (!tarefa) {
    return
  }

  tarefa.prioridade = prioridade

  // atualiza só uma propriedade 
  apiClient.patch(`/tarefas/${tarefa.id}`, JSON.stringify({ prioridade: prioridade.toString() }), {
    headers: {
      "Content-Type": "application/json"
    }
  })
}

// filtrando os tipos de tarefas e ordenando por prioridade
const tarefasNaoConcluidas = computed(() => tarefas.value.filter(tarefa => !tarefa.concluido).sort((a, b) => b.prioridade - a.prioridade))

const tarefasConcluidas = computed(() => tarefas.value.filter(tarefa => tarefa.concluido))

function concluirTarefa(id, concluido) {
  const tarefa = tarefas.value.find(tarefa => tarefa.id === id)
  if (!tarefa) {
    return
  }



 
 
    apiClient.patch(`/tarefas/${tarefa.id}`, JSON.stringify({ concluido: concluido }), {
      headers: {
        "Content-Type": "application/json"
      }
    })
 

}


</script>

<template>
  <div id="flex-container">
    <div id="container">
      <div class="borda esquerda">
        <div class="caixa">
          <!-- formulário -->
          <div class="formulario">
            <form @submit="addTarefa">
              <input type="text" v-model="pegarTarefas">
              <input type="submit" value="enviar">
            </form>
          </div>

          <hr>
          <!-- tarefas não conluídas -->
          <div class="tarefas">
            <div class="tarefa" v-for="tarefa in tarefasNaoConcluidas" :key="tarefa.id">

              <span>{{ tarefa.valor }}</span>
              <select class="prioridade" @change="mudarPrioridade(tarefa.id, $event.target.value)"
                v-model="tarefa.prioridade">
                <option value="1">1</option>
                <option value="2">2</option>
                <option value="3">3</option>
              </select>
              <div class="checkbox">
                <input type="checkbox" v-model="tarefa.concluido" @change="concluirTarefa(tarefa.id, tarefa.concluido)">
                <img src="../public/checked.svg" alt="">
              </div>

              <button @click="excluirTarefa(tarefa.id)">
                <img src="../public/lixo.svg" alt="excluir">
              </button>

            </div>
          </div>
        </div>
      </div>

      <div class="borda direita">
        <div class="caixa">
          <div class="formulario">

            <h4>Tarefas Concluídas</h4>

          </div>
          <hr>

          <!-- tarefas concluídas -->
          <div class="tarefas">
            <div class="tarefa" v-for="tarefa in tarefasConcluidas" :key="tarefa.id">

              <span>{{ tarefa.valor }}</span>

              <div class="checkbox">
                <input type="checkbox" v-model="tarefa.concluido" @change="concluirTarefa(tarefa.id, tarefa.concluido)">
                <img src="../public/checked.svg" alt="">
              </div>

              <button @click="excluirTarefa(tarefa.id)">
                <img src="../public/lixo.svg" alt="excluir">
              </button>

            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped lang="scss">
hr {
  border-top: 1px solid #68378e;
  margin: 12px 0;
}

#flex-container {
  height: 100vh;
  display: flex;
  align-items: center;
  justify-content: center;

  #container {
    background-color: #fff;
    border-radius: 6px;
    max-width: 800px;
    max-height: 700px;
    width: calc(100% - 60px);
    height: calc(100% - 60px);
    display: flex;
    padding: 24px;
    gap: 24px;
    box-shadow: 4px 4px 20px 1px #00000088;

    .esquerda {
      flex: 3;
    }

    .direita {
      flex: 2;
    }

    .borda {
      background: linear-gradient(135deg, #d6aeeb, #68378e);
      height: 100%;
      border-radius: 6px;
      padding: 6px;

      .caixa {
        display: flex;
        flex-direction: column;
        background-color: #fff;
        height: 100%;
        border-radius: 6px;
        padding: 12px;
        position: relative;
      }
    }
  }
}

.formulario {
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
}

form {
  display: flex;
  gap: 6px;
  width: 100%;

  [type="text"] {
    border-radius: 6px;
    height: 30px;
    width: 100%;
    border: 1px solid #68378e;
    padding: 0 6px;

    &:focus {
      outline: 1px solid #68378e;
    }
  }

  [type="submit"] {
    border-radius: 6px;
    padding: 0 6px;
    height: 30px;
    background-color: #d6aeeb;
    color: #68378e;
    border: 1px solid #68378e;
    font-weight: 600;

    &:hover {
      cursor: pointer;
    }
  }
}

.tarefas {
  display: flex;
  flex-direction: column;
  gap: 12px;
  overflow-y: auto;

  .tarefa {
    display: flex;
    border-radius: 4px;
    align-items: center;
    border: 1px solid #68378e;
    border-radius: 4px;
    padding: 6px;
    text-transform: capitalize;
    gap: 4px;

    .prioridade {
      border: 1px solid #68378e;
      border-radius: 4px;
      min-width: 22px;
      width: 22px;
      height: 22px;
      -webkit-appearance: none;
      -moz-appearance: none;
      appearance: none;
      text-indent: 1px;
      text-overflow: '';
      text-align: center
    }

    span {
      width: 100%;
    }

    button {
      height: 22px;
      border: none;
      background: transparent;
      border: 1px solid #68378e;
      border-radius: 4px;
      width: 22px;
      min-width: 22px;

      img {
        object-fit: cover;
        height: 100%;
        width: 100%;
      }

      &:hover {
        cursor: pointer;
      }
    }

    .checkbox {
      height: 22px;
      min-width: 22px;
      width: 22px;
      border: 1px solid #68378e;
      position: relative;
      border-radius: 4px;
      display: flex;
      align-items: center;
      justify-content: center;

      img {
        opacity: 0;
        pointer-events: none;
        height: 18px;
      }

      &:has([type="checkbox"]:checked) {
        background-color: #d6aeeb;

        img {
          opacity: 1;
        }
      }
    }

    [type="checkbox"] {
      height: 100%;
      width: 100%;
      position: absolute;
      left: 0;
      top: 0;
      opacity: 0;
    }

    img {
      height: 24px;
    }
  }
}
</style>
