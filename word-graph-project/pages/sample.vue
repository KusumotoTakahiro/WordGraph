<template>
    <v-row>
    <v-col class="text-center">
        <blockquote class="blockquote">
        <header>
            <small>
            <em>word graph</em>
            </small>
        </header>
        <p>ここにWordGraphを表示する．</p>
        <div id="cy"></div>
        <v-btn @click="update('node')">button</v-btn>
        <footer>
            <small>
            <em>&mdash;word graph</em>
            </small>
        </footer>
        </blockquote>
    </v-col>
    </v-row>
</template>

<script>
import cytoscape from 'cytoscape';
export default {
    name: 'SamplePages',
    data() {
        return {
            count: 0,
            node_id : 1,
            edge_id : 0,
            sita : 0,
            k : 100
        }
    },
    methods: {
        update_nodes : function() {
            this.cy.add([
                { 
                    group : 'nodes',
                    data : { id : this.node_id },
                    position : { x : 150 + Math.cos(this.sita)*this.k, y: 150 + Math.sin(this.sita)*this.k},
                },
            ]);
            this.node_id += 1;
            this.edge_id += 1;
            this.sita += Math.PI/10;
            this.k = 200 + Math.random()*100;
        },
        update_edges : function() {
            this.cy.add({
                group : 'edges',
                data : { 
                    id : this.edge_id, 
                    source : this.node_id-2,
                    target : this.node_id-1,
                },
            })
        },
        update : function() {
            this.update_nodes();
            //this.update_edges();
        },
        view_init : function() {
            this.cy = cytoscape({

            container: document.getElementById('cy'),

            elements: [ // list of graph elements to start with
            ],
            style: [ // the stylesheet for the graph
                {
                    selector: 'node',
                    style: {
                        'background-color': 'brown',
                        'color' : 'red',
                        'label': 'data(id)'
                    }
                },
                {
                    selector: 'edge',
                    style: {
                        'width': 3,
                        'line-color': '#ccc',
                        'target-arrow-color': '#ccc',
                        'target-arrow-shape': 'triangle',
                        'curve-style': 'bezier'
                    }
                }
            ],
            layout: {
                name: 'grid',
                rows: 1
            }
            });
        }
    },
    mounted : function(){
        this.view_init()
    }
}
</script>
<style scoped>
#cy {
    width: 100%;
    height: 80%;
    position: absolute;
    top: 50px;
    left: 0px;
    text-align: left;
}

</style>