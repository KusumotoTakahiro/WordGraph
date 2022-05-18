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
        <button v-on:click="add_node">push</button>
        <div id="cy"></div>
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
        }
    },
    methods: {
        add_node: function() {
            console.info(this.cy)
            this.cy.add([
                {'group': 'nodes', data: {'id': 'node' + this.count }, position: {x: 300, y:200} },
                {'group': 'edges', data: {'id': 'edge' + this.count, 'source': 'node' + this.count, 'target': 'cat'}}
            ])
        },
        view_init: function() {
            this.cy = cytoscape({

    container: document.getElementById('cy'),

  elements: [ // list of graph elements to start with
    { // node a
        data: { id: 'a' }
    },
    { // node b
        data: { id: 'b' }
    },
    { // edge ab
        data: { id: 'ab', source: 'a', target: 'b' }
    }
    ],

  style: [ // the stylesheet for the graph
    {
        selector: 'node',
        style: {
        'background-color': '#666',
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
    mounted: function() {
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