<template>
    <div class="showcase">
        <main>
            <header>
                <h2>Showcase</h2>
            </header>
            <section>
                <button v-for="pokemon_ in pokemonList" :key="pokemon_.url" @click="onClickPokemon(pokemon_.url)" v-b-modal.modal-pokemon-info> {{ pokemon_.name }}</button>
            </section>
        </main>
        <footer>
            <b-pagination @change="onChangePagination($event)" v-model="currentPage" :per-page="1" :total-rows="rows" align="center" size="sm"></b-pagination>
        </footer>

        <b-modal id="modal-pokemon-info" title="Pokemon Basic Info" @ok="onClickOK()" :ok-title="'Add Pokémon'">
            <PokemonBasicInfo  v-if="selectedPokemon" :selectedPokemon="selectedPokemon"></PokemonBasicInfo>
        </b-modal>

    </div>
</template>

<script>
    import axios from 'axios';
    import PokemonBasicInfo from "./PokemonBasicInfo";

    export default {
        name: 'ShowCase',
        components: {
            PokemonBasicInfo
        },
        data() {
            return {
                pokemonList: [],
                currentPage: 1,
                rows: 100,
                limit: 10,
                selectedPokemon: {},
                selectedPokemonList: []
            }
        },
        mounted() {
            this.getPokemons(this.currentPage, this.limit);
        },
        methods: {
            showToast(message, title, variant, append = false) {
                this.$bvToast.toast(message, {
                    title: title,
                    toaster: 'b-toaster-top-center',
                    solid: true,
                    variant: variant,
                    appendToast: append
                })
            },
            getPokemons(pageNumber, limit) {
                axios.get(`https://pokeapi.co/api/v2/pokemon?offset=${pageNumber}&limit=${limit}`).then(response => {
                    this.pokemonList = response.data.results;
                })
            },
            getPokemonDetail(pokemonUrl) {
                this.selectedPokemon = {};
                axios.get(pokemonUrl).then(response => {
                    this.selectedPokemon = response.data;
                })
            },
            isTotalWeightExceeded() {
                if (this.selectedPokemon.weight <= 500) {
                    return this.computeTotatlWeight(this.selectedPokemonList) + this.selectedPokemon.weight > 500;
                } else {
                    return true;
                }
            },
            computeTotatlWeight(list) {
                const reducer = (accumulator, item) => {
                    return accumulator + item.weight;
                };
                return list.reduce(reducer, 0);
            },
            onChangePagination(pageNumber) {
                this.getPokemons(pageNumber, this.limit);
            },
            onClickPokemon(pokemonUrl) {
                this.getPokemonDetail(pokemonUrl);
            },
            onClickOK() {
                if (this.selectedPokemonList.length === 3) {
                    this.showToast('You could select maximum of three!', 'Maximum Warning', 'warning');
                    return;
                }

                if (this.selectedPokemonList.find(item => item.id === this.selectedPokemon.id)) {
                    this.showToast("You couldn't select same Pokémon!", 'Same Pokémon Warning', 'warning');
                    return;
                }

                if (this.isTotalWeightExceeded()) {
                    this.showToast("The total weight of the selected Pokémon shouldn’t be more than 500!", 'Pokémon Weight Limit Warning!', 'warning');
                    return;
                }

                this.selectedPokemonList.push(this.selectedPokemon);

                this.$emit('onAddedPokemon',
                    {selectedPokemonList: this.selectedPokemonList, totalWeight: this.computeTotatlWeight(this.selectedPokemonList)});

                this.showToast("The selected Pokémon added.", 'Pokémon Added', 'success');
            }
        }
    }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
    .showcase {
        flex: 1 1 100%;
        max-width: 25%;
        min-width: 25%;
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        padding: 4px 16px 4px;

        main, header, h2, section, footer, main {
            width: 100%
        }

        h2 {
            margin: 0;
            font-size: 20px;
            font-weight: bold;
        }

        main {
            display: flex;
            flex-direction: column;
        }

        section {
            display: flex;
            flex-flow: row wrap;
            justify-content: space-between;
            align-items: center;
            margin-top: 8px;

            button {
                flex: 1 1 100%;
                max-width: 45%;
                min-width: 45%;
                height: 35px;
                margin-bottom: 16px;
                background-color: #ffffff7d;
                cursor: pointer;
            }
        }

        footer {
            display: flex;
            justify-content: center;
        }
    }
</style>