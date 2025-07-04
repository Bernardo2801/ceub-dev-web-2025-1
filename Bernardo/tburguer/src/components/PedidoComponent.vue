<template>
    <div>
        <!-- Alerta -->
        <mensagem-alerta :texto="mensagem" :tipo="tipoMensagem" :visivel="mostrarMensagem"
            @fechou="mostrarMensagem = false" />

        <form id="pedido-form" @submit="criarPedido($event)">
            <div>
                <p id="nome-hamburguer-content">
                    {{ burguer && burguer.nome ? burguer.nome : "-" }}
                </p>
                <img id="foto-content" :src="burguer && burguer.foto ? burguer.foto : ''" />
            </div>
            <div class="inputs" id="form-pedido">
                <label for="nome_cliente">Nome</label>
                <input type="text" id="nome-cliente" name="nome-cliente" v-model="nomeCliente"
                    placeholder="Digite seu Nome" />
            </div>
            <div class="inputs">
                <label for="ponto-carne">Nível de wasabi</label>
                <select id="ponto-carne" name="ponto-carne" v-model="pontoCarneSelecionado">
                    <option value="" selected>Selecione</option>
                    <option v-for="pontoCarne in listaPontoCarne" :key="pontoCarne.id" :value="pontoCarne">
                        {{ pontoCarne.descricao }}
                    </option>
                </select>
            </div>
            <div id="opcionais-titulo" class="inputs">
                <label for="Complemento">Adicione uma Bebida</label>

                <div class="checkbox-container" v-for="bebida in listaBebidas" :key="bebida.id">
                    <input type="checkbox" :name="bebida.nome" :value="bebida" v-model="listaBebidasSelecionadas" />
                    <span>*{{ bebida.nome }}*</span>
                </div>
            </div>
            <div class="inputs">
                <input type="submit" class="submit-btn" value="Confirmar Pedido" />
            </div>
        </form>
    </div>
</template>

<script>
import MensagemAlerta from "@/components/MensagemAlerta.vue";

export default {
    name: "PedidoComponent",
    props: {
        burguer: null
    },
    components: {
        MensagemAlerta
    },
    data() {
        return {
            nomeCliente: "",
            pontoCarneSelecionado: "",
            listaPontoCarne: [],
            listaComplementos: [],
            listaBebidas: [],
            listaComplementosSelecionados: [],
            listaBebidasSelecionadas: [],
            mensagem: "",
            tipoMensagem: "sucesso",
            mostrarMensagem: false
        };
    },
    methods: {
        exibirMensagem(texto, tipo = 'sucesso') {
            this.mensagem = texto;
            this.tipoMensagem = tipo;
            this.mostrarMensagem = true;
            setTimeout(() => {
                this.mostrarMensagem = false;
            }, 4000);
        },

        async getTipoPontos() {
            const response = await fetch("http://localhost:3000/tipos_pontos");
            const data = await response.json();
            this.listaPontoCarne = data;
        },
        async getOpcionais() {
            const response = await fetch("http://localhost:3000/opcionais");
            const responseJson = await response.json();
            this.listaComplementos = responseJson.complemento;
            this.listaBebidas = responseJson.bebidas;
        },
        async criarPedido(e) {
            e.preventDefault();

            if (!this.nomeCliente || !this.pontoCarneSelecionado) {
                this.exibirMensagem("Por favor, preencha seu nome e nível de wasabi.", "erro");
                return;
            }

            const dadosPedido = {
                nome: this.nomeCliente,
                ponto: this.pontoCarneSelecionado,
                bebidas: Array.from(this.listaBebidasSelecionadas),
                complementos: Array.from(this.listaComplementosSelecionados),
                statusId: 5,
                hamburguer: this.burguer
            };

            try {
                const response = await fetch("http://localhost:3000/pedidos", {
                    method: "POST",
                    headers: { "Content-Type": "application/json" },
                    body: JSON.stringify(dadosPedido)
                });

                if (response.ok) {
                    this.exibirMensagem("Pedido criado com sucesso!", "sucesso");
                    setTimeout(() => {
                        this.$router.push('/menu');
                    }, 1000);
                } else {
                    this.exibirMensagem("Erro ao criar o pedido. Tente novamente.", "erro");
                }
            } catch (error) {
                this.exibirMensagem("Erro ao comunicar com o servidor.", "erro");
                console.error(error);
            }
        }
    },
    mounted() {
        this.getTipoPontos();
        this.getOpcionais();
    }
};
</script>

<style scoped>
#foto-content {
    margin-bottom: 16px;
    border-radius: 16px;
    position: relative;
    z-index: -1;
    justify-content: center;
    width: 100%;
    height: 180px;
    object-fit: cover;
}

#nome-hamburguer-content {
    font-size: 43px;
    font-weight: bold;
    text-align: start;
    margin-bottom: -90px;
    margin-left: 40px;
    color: antiquewhite;
    padding: 16px;
}

#pedido-form {
    max-width: 750px;
    margin: 0 auto;
}

.inputs {
    display: flex;
    flex-direction: column;
    margin-bottom: 16px;
}

label {
    font-weight: bold;
    margin-bottom: 16px;
    color: #222;
    padding: 5px 12px;
    border-left: 4px solid darkgoldenrod;
    display: flex;
}

input,
select {
    padding: 12px;
    width: 300px;
    border: #222 solid 1px;
    border-radius: 8px;
    height: 20px;
    font-size: 12px;
}

select {
    height: 50px;
}

#opcionais-titulo {
    width: 100%;
}

#opcionais-subtitulo {
    font-weight: bold;
    color: #222;
    border-left: 4px solid darkslategrey;
}

.checkbox-container {
    display: flex;
    align-items: flex-start;
    align-content: center;
    width: 100%;
    margin-bottom: 16px;
}

.checkbox-container span,
.checkbox-container input {
    width: auto;
    height: 20px;
}

.checkbox-container span {
    margin-left: 6px;
    font-weight: bold;
}

.submit-btn {
    background-color: #222;
    color: darkgoldenrod;
    font-weight: bold;
    border: solid 2px darkgoldenrod;
    border-radius: 8px;
    cursor: pointer;
    padding: 12px;
    margin: 0 auto;
    font-size: 16px;
    width: 100%;
    height: auto;
    transition: 0.5s;
}

.submit-btn:hover {
    background-color: transparent;
    color: #222;
}
</style>
