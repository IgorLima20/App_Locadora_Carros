<template>
    <div class="container">
        <div class="row justify-content-center">
            <div class="col-md-8">

                <card-component titulo="Busca de marcas">
                    <template v-slot:conteudo>
                        <div class="form-row">
                            <div class="col mb-3">
                                <input-container-component id="inputId" titulo="ID" id-help="idHelp" texto-ajuda="Opcional. Informe o Id da marca">
                                    <input type="number" class="form-control" id="inputId" aria-describedby="idHelp" placeholder="ID" v-model="busca.id">
                                </input-container-component>
                            </div>
                            <div class="col mb-3">
                                <input-container-component id="inputNome" titulo="Nome da Marca" id-help="nomeHelp" texto-ajuda="Opcional. Informe o nome da marca">
                                    <input type="text" class="form-control" id="inputNome" aria-describedby="nomeHelp" placeholder="Nome da Marca" v-model="busca.nome">
                                </input-container-component>
                            </div>
                        </div>
                    </template>        
                    <template v-slot:rodape>
                        <button type="submit" class="btn btn-primary btn-sm float-right" @click="pesquisar()">Pesquisar</button>
                    </template>            
                </card-component>

                <card-component titulo="Relação de Marcas">
                    <template v-slot:conteudo>
                        <table-component 
                        :dados="marcas.data" 
                        :visualizar="{visivel: true, dataToggle: 'modal', dataTarget: '#modalMarcaVisualizar'}" 
                        :atualizar="{visivel: true, dataToggle: 'modal', dataTarget: '#modalMarcaAtualizar'}" 
                        :remover="{visivel: true, dataToggle: 'modal', dataTarget: '#modalMarcaRemover'}" 
                        :titulos="{
                            id: {titulo: 'ID', tipo:'text'},
                            nome: {titulo: 'Nome', tipo:'text'},
                            // imagem: {tiutlo: 'Imagem', tipo:'image'},
                            created_at: {titulo: 'Data de Criação', tipo:'date'},
                        }"
                        ></table-component>
                    </template>        
                    <template v-slot:rodape>
                        <div class="row">
                            <div class="col-10">
                                <paginate-component>
                                    <li v-for="l, key in marcas.links" :key="key" :class="l.active ? 'page-item active' : 'page-item'" @click="paginacao(l)">
                                        <a class="page-link " v-html="l.label"></a>
                                    </li>
                                </paginate-component>
                            </div>
                            <div class="col">
                                 <button type="submit" class="btn btn-primary btn-sm float-right" data-toggle="modal" data-target="#modalMarca">Adicionar</button>
                            </div>
                        </div>
                    </template>            
                </card-component>
            </div>

            <!-- Modal Adicionar -->
            <modal-component id="modalMarca" titulo="Adicionar Marca">
                <template v-slot:alertas>
                    <alert-component tipo="success" :detalhes="transacaoDetalhes" titulo="Cadastro realizado com sucesso" v-if="transacaoStatus == 'Adicionado'"></alert-component>
                    <alert-component tipo="danger" :detalhes="transacaoDetalhes" titulo="Erro ao tentar cadastrar a Marca" v-if="transacaoStatus == 'Erro'"></alert-component>
                </template>

                <template v-slot:conteudo>
                    <div class="form-group">
                        <input-container-component id="novoNome" titulo="Nome da Marca" id-help="novoNomeHelp" texto-ajuda="Informe o nome da marca">
                            <input type="text" class="form-control" id="novoNome" aria-describedby="novoNomeHelp" placeholder="Nome da Marca" v-model="nomeMarca">
                        </input-container-component>

                        <input-container-component id="novoImagem" titulo="Imagem" id-help="novoImagemHelp" texto-ajuda="Selecione uma Imagem PNG">
                            <input type="file" class="form-control-file" id="novoImagem" aria-describedby="novoImagemHelp" placeholder="Selecione uma Imagem" @change="carregarImagem($event)">
                        </input-container-component>
                    </div>
                </template>
                <template v-slot:rodape>
                    <button type="button" class="btn btn-secondary" data-dismiss="modal">Fechar</button>
                    <button type="button" class="btn btn-primary" @click="salvar()">Salvar</button>
                </template>
            </modal-component>

            <!-- Modal Visualizar -->
            <modal-component id="modalMarcaVisualizar" titulo="Visualizar Marca">
                <template v-slot:alertas>
                    <alert-component tipo="success" :detalhes="transacaoDetalhes" titulo="Cadastro realizado com sucesso" v-if="transacaoStatus == 'Adicionado'"></alert-component>
                    <alert-component tipo="danger" :detalhes="transacaoDetalhes" titulo="Erro ao tentar cadastrar a Marca" v-if="transacaoStatus == 'Erro'"></alert-component>
                </template>

                <template v-slot:conteudo>
                    <div class="form-group">
                        <input-container-component titulo="ID">
                            <input type="text" class="form-control" :value="$store.state.item.id" disabled>
                        </input-container-component>

                        <input-container-component titulo="Marca">
                            <input type="text" class="form-control" :value="$store.state.item.nome" disabled>
                        </input-container-component>

                        <input-container-component titulo="Data de Criação">
                            <input type="text" class="form-control" :value="$store.state.item.created_at" disabled>
                        </input-container-component>
                    </div>
                </template>
                <template v-slot:rodape>
                    <button type="button" class="btn btn-secondary" data-dismiss="modal">Fechar</button>
                </template>
            </modal-component>

            <!-- Modal Remover -->
            <modal-component id="modalMarcaRemover" titulo="Remover Marca">
                <template v-slot:alertas>
                    <alert-component tipo="success" :detalhes="$store.state.transacao" titulo="Transação realizada com sucesso" v-if="$store.state.transacao.status == 'sucesso'"></alert-component>
                    <alert-component tipo="danger"  :detalhes="$store.state.transacao" titulo="Erro na transação" v-if="$store.state.transacao.status == 'erro'"></alert-component>
                </template>

                <template v-slot:conteudo v-if="$store.state.transacao.status != 'sucesso'">
                    <div class="form-group">
                        <input-container-component titulo="ID">
                            <input type="text" class="form-control" :value="$store.state.item.id" disabled>
                        </input-container-component>

                        <input-container-component titulo="Marca">
                            <input type="text" class="form-control" :value="$store.state.item.nome" disabled>
                        </input-container-component>
                    </div>
                </template>
                <template v-slot:rodape>
                    <button type="button" class="btn btn-secondary" data-dismiss="modal">Fechar</button>
                    <button type="button" class="btn btn-danger" @click="remover()"  v-if="$store.state.transacao.status != 'sucesso'">Remover</button>
                </template>
            </modal-component>

            <!-- Modal Atualizar -->
            <modal-component id="modalMarcaAtualizar" titulo="Atualizar Marca">
                <template v-slot:alertas>
                    <alert-component tipo="success" :detalhes="$store.state.transacao" titulo="Transação realizada com sucesso" v-if="$store.state.transacao.status == 'sucesso'"></alert-component>
                    <alert-component tipo="danger"  :detalhes="$store.state.transacao" titulo="Erro na transação" v-if="$store.state.transacao.status == 'erro'"></alert-component>
                </template>

                <template v-slot:conteudo>
                    <div class="form-group">
                        <input-container-component id="atualizarNome" titulo="Nome da Marca" id-help="atualizarNomeHelp" texto-ajuda="Informe o nome da marca">
                            <input type="text" class="form-control" id="atualizarNome" aria-describedby="atualizarNomeHelp" placeholder="Nome da Marca" v-model="$store.state.item.nome">
                        </input-container-component>

                        <input-container-component id="atualizarImagem" titulo="Imagem" id-help="atualizarImagemHelp" texto-ajuda="Selecione uma Imagem PNG">
                            <input type="file" class="form-control-file" id="atualizarImagem" aria-describedby="atualizarImagemHelp" placeholder="Selecione uma Imagem" @change="carregarImagem($event)">
                        </input-container-component>
                    </div>
                </template>
                <template v-slot:rodape>
                    <button type="button" class="btn btn-secondary" data-dismiss="modal">Fechar</button>
                    <button type="button" class="btn btn-danger" @click="atualizar()">Atualizar</button>
                </template>
            </modal-component>
        </div>
    </div>
</template>

<script>
    export default {
        data() {
            return {
                urlBase: 'http://127.0.0.1:8000/api/v1/marca',
                urlPaginacao: '',
                urlFiltro: '',
                nomeMarca: '',
                arquivoImagem: [],
                transacaoStatus: '',
                transacaoDetalhes: {},
                marcas: { data: []},
                busca: { id:'', nome:'' }
            }
        },
        methods: {
            atualizar(){
                let formData = new FormData();
                formData.append('_method', 'patch');
                formData.append('nome', this.$store.state.item.nome);
                if (this.arquivoImagem[0]) {
                    formData.append('imagem', this.arquivoImagem[0]);
                }
                let config = {
                        headers: {
                            'Content-Type': 'multipart/form-data'
                        }
                    };
                let url = this.urlBase + '/' + this.$store.state.item.id;
                axios.post(url, formData, config)
                .then(response => {
                    atualizarImagem.value = '';
                    this.$store.state.transacao.status = 'sucesso';
                    this.$store.state.transacao.mensagem = 'Marca atualiza com sucesso!!';
                    this.carregarLista();
                }).catch(errors => {
                    this.$store.state.transacao.status = 'erro';
                    this.$store.state.transacao.mensagem = errors.response.data.message;
                    this.$store.state.transacao.dados = errors.response.data.errors;
                });
            },
            remover(){
                let confirmacao = confirm('Tem certeza que deseja remover esse registro ?');
                if (confirmacao) {
                    let formData = new FormData();
                    formData.append('_method', 'delete');
                    let url = this.urlBase + '/' + this.$store.state.item.id;
                    axios.post(url, formData)
                        .then(response => {
                            this.$store.state.transacao.status = 'sucesso';
                            this.$store.state.transacao.mensagem = response.data.msg;
                            this.carregarLista();
                        }).catch(errors => {
                            this.$store.state.transacao.status = 'erro';
                            this.$store.state.transacao.mensagem = response.data.msg;
                        });
                } else {
                    return false;
                }
            },
            pesquisar(){
                let filtro = '';
                for(let chave in this.busca) {
                    if (this.busca[chave]) {
                        if (filtro != '') {
                            filtro += ';';
                        }
                        filtro += chave + ':like:' + this.busca[chave];
                    }
                }
                if (filtro != '') {
                    this.urlPaginacao = 'page=1';
                    this.urlFiltro = '&filtro=' + filtro;
                } else {
                    this.urlFiltro = '';
                }
                this.carregarLista();
            },
            paginacao(l) {
                if (l.url) {
                    this.urlPaginacao = l.url.split('?')[1];
                    this.carregarLista();
                }
            },
            carregarLista() {
                let url = this.urlBase + '?' + this.urlPaginacao + this.urlFiltro;
                axios.get(url)
                .then(response => {
                    this.marcas = response.data;
                }).catch(errors => {
                    console.log(errors);
                })
            },
            carregarImagem(e) {
                this.arquivoImagem = e.target.files;
            },
            salvar() {
                let frm = new FormData();
                frm.append('nome', this.nomeMarca);
                frm.append('imagem', this.arquivoImagem[0]);
                let config = {
                    headers: {
                        'Content-Type': 'multipart/form-data',
                    }
                };
                axios.post(this.urlBase, frm, config)
                .then(response => {
                    this.transacaoStatus = 'Adicionado';
                    this.transacaoDetalhes = {
                        mensagem: 'ID do Registro: '+detalhes.data.id

                    };
                })
                .catch(errors => {
                    this.transacaoStatus = 'Erro';
                    this.transacaoDetalhes = {
                        mensagem: errors.response.data.message,
                        dados: errors.response.data.errors
                    };
                })
            }
        },
        mounted() {
            this.carregarLista();
        }
    }
</script>