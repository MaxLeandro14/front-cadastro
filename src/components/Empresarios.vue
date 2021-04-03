<template>
  <div>
    <header class="header_area">
      <h1>Cadastro de Empresários</h1>
    </header>
    <main class="cadastro">
      <form v-on:submit.prevent="cadastro">
        <input name="nome" v-model="dados.nome" type="text" placeholder="- Nome Completo"/>
        <input name="telefone" v-model="dados.telefone" type="text" placeholder="Celular"/>
        <select class="select" v-model="dados.estado" @change="onChangeEstado" >
          <option value="null" disabled>Selecione Estado</option>
          <option value="AC">Acre</option>
          <option value="AL">Alagoas</option>
          <option value="AP">Amapá</option>
          <option value="AM">Amazonas</option>
          <option value="BA">Bahia</option>
          <option value="CE">Ceará</option>
          <option value="DF">Distrito Federal</option>
          <option value="ES">Espírito Santo</option>
          <option value="GO">Goiás</option>
          <option value="MA">Maranhão</option>
          <option value="MT">Mato Grosso</option>
          <option value="MS">Mato Grosso do Sul</option>
          <option value="MG">Minas Gerais</option>
          <option value="PA">Pará</option>
          <option value="PB">Paraíba</option>
          <option value="PR">Paraná</option>
          <option value="PE">Pernambuco</option>
          <option value="PI">Piauí</option>
          <option value="RJ">Rio de Janeiro</option>
          <option value="RN">Rio Grande do Norte</option>
          <option value="RS">Rio Grande do Sul</option>
          <option value="RO">Rondônia</option>
          <option value="RR">Roraima</option>
          <option value="SC">Santa Catarina</option>
          <option value="SP">São Paulo</option>
          <option value="SE">Sergipe</option>
          <option value="TO">Tocantins</option>
        </select>

        <select class="select" v-model="dados.cidade">
          <option :value="null" disabled>Selecione Cidade</option>
          <option v-for="(cidade, key) in cidades" :key="key" :value="cidade.nome">{{ cidade.nome }}</option>
        </select>

        <select class="select" v-model="dados.pai_empresarial">
          <option :value="null" disabled>Selecione Pai Empresarial</option>
          <option v-for="(pai, key) in empresarios" :key="key" :value="{ id: pai.id, text: pai.nome }">{{ pai.nome }}</option>
        </select>
        <div v-if="erros">
          <div class="list-erro">
            <ul>
              <li v-for="(erro, i) in erros" :key="i">{{erro}}</li>
            </ul>
          </div>
        </div>
        <button type="submit" class="btn btn-s">Cadastrar</button>
      </form>
    </main>
    <div class="box-table">
      <table class="tab-table">
        <tbody>
          <tr>
            <th>Nome Completo</th>
            <th>Celular</th>
            <th>Cidade/UF </th>
            <th>Cadastrado em</th>
            <th>Pai Empresarial</th>
            <th>Rede</th>
            <th></th>
          </tr>
          <tr v-for="(empresario, key) in empresarios" :key="key">
            <td>{{empresario.nome}}</td>
            <td>{{empresario.telefone}}</td>
            <td>{{empresario.cidade}}</td>
            <td>{{empresario.created_at}}</td>
            <td>{{empresario.nome_pai}}</td>
            <td><button class="btn btn-r" @click="verRede(empresario.id)">Rede</button></td>
            <td><button class="btn btn-e" @click="deleteEmpre(empresario.id, key)">Excluir</button></td>
          </tr>
        </tbody>
      </table>
    </div>
       <!-- modal -->
    <div class="modal visible" v-if="modalAtivar">
      <div class="modal__content">
        <label for="title"> Rede </label>
      </div>
      <div class="modal__actions">
        <p>Os dados com os filhos estão chegando, verifique no console. </p>
        <p>Não coloquei para exibir essa parte :) . </p>
        <button class="btn btn--passive" @click="modalAtivar = false"> Fechar </button>
      </div>
    </div>
    <!-- fim modal -->
  </div>
</template>

<script>
import axios from 'axios'
export default {
  name: 'Empresarios',
  props: {
  },
  data () {
    return {
      dados: {
        nome: '',
        telefone: null,
        estado: null,
        cidade: null,
        nome_pai: null,
        pai_empresarial:  null
      },
      cidades: [],
      empresarios: [],
      modalAtivar: false,
      filhos: [],
      erros: ''
    }
  },
  mounted () {
    this.getList()
  },
  methods: {
    getList: async function () {
      let self = this
      await axios.get('https://stormy-lowlands-89661.herokuapp.com/api/empresarios')
        .then(res => {
         self.empresarios = [...res.data]
        })
        .catch(error => console.log(error))
    },
    cadastro: async function () {
      let self = this
      this.erros = ''
      await axios.post('https://stormy-lowlands-89661.herokuapp.com/api/empresarios', {
        nome: this.dados.nome,
        estado: this.dados.estado,
        cidade: this.dados.cidade,
        telefone: this.dados.telefone,
        nome_pai: (this.dados.pai_empresarial !== null) ? this.dados.pai_empresarial.text : '',
        pai_empresarial: (this.dados.pai_empresarial !== null) ? parseInt(this.dados.pai_empresarial.id) : ''
      })
        .then(res => {
          if (res.data[0] === '0') {
            self.erros = res.data[1]
  
          }else{
            self.getList()          
            self.dados = {
                nome: '',
                telefone: null,
                estado: null,
                cidade: null,
                nome_pai: null,
                pai_empresarial: null
              }
              self.erros = ''
          }

          
        })
    },
    onChangeEstado: async function(){
      this.dados.cidade = null
      this.cidades = []
       let self = this
       await axios.get('https://servicodados.ibge.gov.br/api/v1/localidades/estados/' + this.dados.estado + '/municipios')
        .then(res => {
          self.cidades = [...res.data]
        })
        .catch(() => {
        })
    },
    deleteEmpre: async function (id, i) {
      if(confirm("Você realmente quer deletar?")){
        let self = this
        await axios.delete('https://stormy-lowlands-89661.herokuapp.com/api/empresarios/' + id)
        .then(res => {
          // remover o item do array
          if (res.data == "ok"){
              self.empresarios.forEach(function(el, index) {
                if(index === i)
                  self.empresarios.splice(index, 1);
              })
          }

        })
      }
    },
    verRede: async function (id) {
      this.modalAtivar = true
      let self = this
      await axios.get('https://stormy-lowlands-89661.herokuapp.com/api/empresarios/rede/' + id)
        .then(res => {
          console.log(res.data)
          self.filhos = res.data
        })
        .catch(() => {
        })
    }
  }
}
</script>

<style scoped>
div, h1, select, input {
  padding: 0px;
  margin: 0px;
}
body {
    margin: 0;
}
.header_area {
  background: #41b776;
  padding: 14px 5px;
  color: #fff;
}
.cadastro {
  padding: 46px 5px 5px 5px;
  margin: 0 auto;
  max-width: 300px;
}
input {
  outline: none;
  -webkit-box-sizing: border-box;
    -moz-box-sizing: border-box;
    box-sizing: border-box;
}
select, input {
  margin: 5px 0px;
  padding: 9px;
  border: 1px solid #00843b;
  border-radius: 5px;
  width: 100%;
  outline: none;
  font-size: 16px;
}

select {
    width: 100%;
}
.select {
  cursor: pointer;
  line-height: 1.1;
}
@media only screen (max-width: 500px) {
  .cadastro {
    max-width: 100%;
  }
}
.list-erro {
  padding: 5px;
  background: #f5f5f5;
  margin: 5px;
}
.list-erro ul {
  list-style: none;
  margin: 0;
  text-align: left;
  padding: 2px 4px;
}
.list-erro li {
  margin: 5px 0px;
  color: #00843b;
}
/* table*/
.box-table {
  margin: 45px auto;
  max-width: 1100px;
}
.tab-table {
   display: block;
   font-family: sans-serif;
   -webkit-font-smoothing: antialiased;
   font-size: 16px;
   overflow: auto;
   width: auto;
}
 .tab-table th {
   background-color: #429088;
   color: white;
   font-weight: normal;
   padding: 20px 30px;
   text-align: center;
}
 .tab-table td {
   background-color: #eee;
   color: #6f6f6f;
   padding: 20px 30px;
}
.btn {
  padding: 10px 5px;
  border-radius: 5px;
  border: 1px solid #70c469;
  cursor: pointer;
  width: 100%;
  color: #fff;
  font-weight: 600;
}
.btn-s {
  background: #69c4bb;
}
.btn-e {
  background: #ef0546;
}
.btn-r {
  background: #f3c900;
}
.btn:hover {
  background: #96ce91;
}
/* modal */
.modal {
  background-color: #FDFFFC;
  border-radius: 10px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
  display: none;
  left: 10%;
  position: fixed;
  top: 30vh;
  width: 80%;
  z-index: 100;
}

.modal.visible {
  animation: fade-slide-in 0.3s ease-out forwards;
  display: block;
}

.modal .modal__title {
  background-color: #FFB7B7;
  border-bottom: 1px solid #FFB7B7;
  border-radius: 10px 10px 0 0;
  color: #FDFFFC;
  font-family: "Chango", cursive;
  margin: 0;
  padding: 1rem;
  text-transform: uppercase;
}

.modal .modal__content {
  padding: 1rem;
}

.modal .modal__actions {
  display: flex;
  justify-content: flex-end;
  padding: 1rem;
}

#add-modal .modal__content {
  display: flex;
  flex-direction: column;
}

.modal label {
  color: #2A2A2A;
  font-weight: bold;
  margin: 0.5rem 0;
}
.modal .btn {
  background-color: #429088;
  border: 1px solid #E8E8E8;
  border-radius: 6px;
  box-shadow: 0 1px 8px rgba(0, 0, 0, 0.26);
  color: #FFFFFF;
  cursor: pointer;
  margin: 0 0.5rem;
  padding: 0.5rem 1.5rem;
}

.btn--passive {
  background-color: transparent;
  border: none;
    box-shadow: none;
  color: #2A2A2A;
}

.btn--passive:hover,
.btn--passive:active {
  background-color: #41b776;
}

.btn--success{
  background-color: #B8FFD2;
  border-color: #B8FFD2;
  color: #2A2A2A;
}

.btn--success:hover,
.btn--success:active {
    background-color: #41b776;
    border-color: #92E7B1;
}

@media (min-width: 768px) {
  .modal {
    width: 40rem;
    left: calc(50% - 20rem);
  }
}

@keyframes fade-slide-in {
  from {
    transform: translateY(-5rem);
  }
  to {
    transform: translateY(0);
  }
}
</style>
