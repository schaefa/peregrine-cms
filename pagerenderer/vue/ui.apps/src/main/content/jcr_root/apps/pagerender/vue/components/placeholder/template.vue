<!--
  #%L
  peregrine vuejs page renderer - UI Apps
  %%
  Copyright (C) 2017 headwire inc.
  %%
  Licensed to the Apache Software Foundation (ASF) under one
  or more contributor license agreements.  See the NOTICE file
  distributed with this work for additional information
  regarding copyright ownership.  The ASF licenses this file
  to you under the Apache License, Version 2.0 (the
  "License"); you may not use this file except in compliance
  with the License.  You may obtain a copy of the License at
  
  http://www.apache.org/licenses/LICENSE-2.0
  
  Unless required by applicable law or agreed to in writing,
  software distributed under the License is distributed on an
  "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
  KIND, either express or implied.  See the License for the
  specific language governing permissions and limitations
  under the License.
  #L%
  -->
<template>
    <div v-if="isEditMode" class="per-drop-target"
        v-on:allowDrop="allowDrop" v-on:drop="drop" v-bind:data-per-path="model.path" data-per-droptarget="true" v-bind:data-per-location="model.location">
        {{componentName}}
    </div>
</template>

<script>
export default {
    props: ['model'],
    computed: {
        isEditMode: function() {
            if(window.parent) {
                if(window.parent.$perAdminApp && window.parent !== window) {
                    return this.$root.view !== 'preview'
                }
            }
            return false
        },
        componentName: function() {
            let post = ''
            if(this.model.location === 'before') post = ' start'
            if(this.model.location === 'after') post = ' end'
            return this.model.component.split('-').pop() + post
        }
    },
    methods: {
        allowDrop: function(e) {
            e.preventDefault()
        },
        drop: function(e) {
            alert('component drop')
        },
        edit: function(e) {
            alert('edit')
        }
    }
}
</script>

<style>
    .per-drop-target {
        border: 1px solid #c0c0c0;
        clear: both;
        padding: 4px;
        margin: 4px;
        text-align: center;
        width: calc(100% - 8px);
        white-space: nowrap;
        text-overflow: ellipsis;
        overflow: hidden;
    }
</style>
