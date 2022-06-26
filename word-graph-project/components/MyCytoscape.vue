<template>
    <div id="cy"></div> 
</template>
<script>
import cytoscape from 'cytoscape';

export default {
    data() {
        return {
            keywords : [],
            theta : 0,
            k : 100,
        }
    },
    methods: {
        init_graph() {
            this.cy = cytoscape({
                container : document.getElementById('cy'),
                elements : [
                    {
                        data : { id : 'start' },
                    },
                ],
                style: [
                    {
                        selector: 'node',
                        style: {
                            'background-color': '#3cb371',
                            'color': 'white',
                            'label' : 'data(id)',
                            //'text-valign': 'center',
                        }
                    },
                    {
                        selector: 'edge',
                        style: {
                            'width':3,
                            'line-color':'#ccc',
                            'target-arrow-color':'#ccc',
                            'target-arrow-shape':'triangle',
                            'curve-style': 'bezier',
                        }
                    }
                ],
                layout: {
                    name: 'grid',
                    rows: 1
                }
            });
        },
        update_nodes(keyword) {
            this.cy.add([
                {
                    group : 'nodes',
                    data : { id : keyword },
                    position : {  
                        x : 150 + Math.cos(this.theta)*this.k, 
                        y : 150 + Math.sin(this.theta)*this.k 
                    }
                }
            ]);
            this.k = 200 + Math.random()*100;
            this.theta += Math.PI/10;
        },
        update_edges(source_node, target_node) {
            this.cy.add([
                {
                    group : 'edges',
                    data : {
                        source : source_node,
                        target : target_node,
                    }
                }
            ]);
        },
        update_graph() {
            let vm = this;
            //先にキーワードをノードとして追加する
            vm.keywords.forEach(function(value){
                if (value.isInGraph==false){
                    vm.update_nodes(value.keyword);
                    value.isInGraph = true;
                }
            });
            //keywordsからedgeを生成する
            vm.keywords.forEach(function(value){
                let before_next = value.before_next;
                let after_next = value.after_next;
                let keyword = value.keyword;
                for (let i = 0; i < before_next.length; i++) {
                    let is_drawn = false;
                    //描画済みのedgeではないか確認する
                    for (let j = 0; j < after_next.length; j++) {
                        if (before_next[i]==after_next[j]){
                            is_drawn = true;
                        }
                    }
                    if (is_drawn==false) {
                        vm.update_edges(keyword, before_next[i]);
                    }
                }
                //before_nextをafter_nextに更新する.重複する分は今のところ無視
                value.after_next = after_next.concat(before_next);
                value.before_next = [];
            })
        },
        zoom_func() {
            let client_w = document.getElementById('cy').clientWidth;
            let client_h = document.getElementById('cy').clientHeight;
            this.cy.zoom({
                level : 2.0,
                renderedPosition: {x: client_w/2, y: client_h/2.5},
            })
        }
    },
    mounted() {
        this.init_graph();
    },
}
</script>

<style scoped>
#cy {
    background-color: darkblue;
    height: calc(100vw / 3);
}
</style>
