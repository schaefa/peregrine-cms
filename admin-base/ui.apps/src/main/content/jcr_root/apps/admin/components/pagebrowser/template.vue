<!--
  #%L
  admin base - UI Apps
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
    <div id="pageBrowserModal" class="pathbrowser pagebrowser modal default modal-fixed-footer">
        <ul ref="pbtabs" class="pathbrowser-tabs">
            <li class="tab">
                <a href="#" :class="tab === 'browse' ? 'active' : ''" v-on:click="select('browse')">
                    <i class="material-icons">list</i>
                </a>
            </li>
            <li class="tab">
                <a href="#" :class="tab === 'cards' ? 'active' : ''" v-on:click="select('cards')">
                    <i class="material-icons">view_module</i>
                </a>
            </li>
            <li class="tab" v-if="withLinkTab">
                <a href="#" :class="tab === 'link' ? 'active' : ''" v-on:click="select('link')">
                    <i class="material-icons">link</i>
                </a>
            </li>
            <li 
                class="indicator" 
                :style="`transform: translateX(${tabIndicatorPosition}px)`">
            </li>
        </ul>
        <div class="pathbrowser-filter" :style="`width: calc(100% - ${searchTabOffset}px)`">
            <input placeholder="search" 
                    type="search" 
                    v-model="search" />
        </div>
        <div class="modal-content">
            <div class="col-browse"> 
                <div v-if="search">
                    <table >
                        <thead>
                        <tr>
                            <th>Name</th>
                            <th>Path</th>
                        </tr>
                        </thead>

                        <tbody>
                        <tr v-for="item in nodes.children" v-if="searchFilter(item)">
                            <td>{{item.name}}</td>
                            <td>{{item.path}}</td>
                        </tr>
                        </tbody>
                    </table>
                </div>
                <template v-if="tab === 'browse' && !search">
                    <span class="current-folder">
                        <button 
                            :disabled="path === '/content'" 
                            type="button" 
                            class="btn-flat" 
                            v-on:click.stop.prevent="selectParent">
                            <i class="material-icons">keyboard_arrow_left</i> 
                        </button>
                        {{path}} ({{list.length}})
                    </span>
                    <ul class="browse-list">
                        <li v-if="isFolder(item)" 
                            v-for="(item, index) in nodes.children" 
                            :class="isSelected(item.path) ? 'selected' : ''"
                            v-on:click.stop.prevent="selectFolder(item)">
                            <input name="selectedItem" type="radio" class="with-gap" :checked="isSelected(item.path)" />
                            <label v-on:click.stop.prevent="selectItem(item.path)"></label>
                            <i class="material-icons">folder</i>
                            <span>{{item.name}}</span>
                        </li>
                    </ul>
                </template>
                <template v-if="tab === 'cards' && !search">
                    <template v-if="list.length > 0">
                        <ul class="cards-toolbar sort-nav">
                            <li>
                                <span class="cards-toolbar-title">Sort</span>
                            </li>
                            <li>
                                <input 
                                    name="pagebrowser_sort_cards" 
                                    type="radio" 
                                    class="with-gap" 
                                    id="pagebrowser_sort_cards_name" 
                                    :checked="sortBy === 'name'"/>
                                <label v-on:click="onSort('name')" for="pagebrowser_sort_cards_name">name</label>
                            </li>
                            <li>
                                <input 
                                    name="pagebrowser_sort_cards" 
                                    type="radio" 
                                    class="with-gap" 
                                    id="pagebrowser_sort_cards_date" 
                                    :checked="sortBy === 'created'"/>
                                <label v-on:click="onSort('created')" for="pagebrowser_sort_cards_date">date</label>
                            </li>
                        </ul>
                    
                        <p class="range-field">
                            <input 
                                type="range" 
                                min="120" 
                                max="400" 
                                v-model="cardSize"/>
                        </p>
                        <!--<admin-components-spinner 
                            v-if="isotopeLoading"
                            width="60" 
                            position="center">
                        </admin-components-spinner> -->
                        
                        <isotope 
                            ref="isotope" 
                            class="isotopes" 
                            v-bind:options="getIsotopeOptions()"
                            v-images-loaded:on="getImagesLoadedCbs()" 
                            v-bind:list="list">
                            <div 
                                v-for="(item, index) in list" 
                                :key="item.path">
                                <div 
                                    v-if="isFolder(item)" 
                                    class="item-folder"
                                    v-bind:style="`width: ${cardSize}px; height: ${cardSize}px`"
                                    v-on:click.stop.prevent="selectFolder(item)">
                                        <div class="item-content">
                                            <i 
                                                class="material-icons"
                                                :style="`font-size: ${cardIconSize(cardSize)}px`">folder_open</i>
                                            <br/>{{item.name}}
                                        </div>
                                </div>
                            </div>
                        </isotope>
                    </template>
                    <p v-else class="flow-text">This folder is empty.</p>
                </template>
                <template v-if="withLinkTab && tab === 'link' && !search">
                    <vue-form-generator
                        v-bind:schema  = "linkSchema"
                        v-bind:model   = "linkModel"
                        v-bind:options = "linkFormOptions">
                    </vue-form-generator>
                </template>
            </div>
            <div class="col-preview">
                <template v-if="preview">
                    <div v-if="isFolder(preview)" class="preview-folder">
                        <i class="material-icons">folder_open</i>
                    </div>
                    <img v-else class="preview-image" v-bind:src="preview.path">
                    <dl class="preview-data">
                        <dt>Name</dt>
                        <dd>{{preview.name}}</dd>
                        <dt>Type</dt>
                        <dd>{{preview.resourceType}}</dd>
                        <dt>Path</dt>
                        <dd>{{preview.path}}</dd>
                        <dt>Created</dt>
                        <dd>{{preview.created}}</dd>
                    </dl>
                </template>
                <div v-else class="no-asset-selected">
                    <span>{{ $i18n('no asset selected') }}</span>
                    <i class="material-icons">info</i>
                </div>
            </div>
        </div>
        <div class="modal-footer">
            <span class="selected-path">{{selectedPath}}</span>
            <button 
                v-if="withLinkTab"
                v-on:click="onUnlink"
                class="modal-action modal-close waves-effect waves-light btn-flat">unlink</button>
            <button 
                class="modal-action modal-close waves-effect waves-light btn-flat">cancel</button>
            <button 
                class="modal-action modal-close waves-effect waves-light btn-flat">select</button>
        </div>
    </div>
</template>

<script>
    export default {
        props: ['model'],
        data: function() {
            return {
                tab: 'browse',
                cardSize: 120,
                search: '',
                preview: '', 
                linkModel: {
                    url: '',
                    newWindow: false
                },
                linkSchema: {
                    fields: [
                        {
                            type: "input",
                            inputType: "text",
                            label: "Url",
                            model: "url",
                            placeholder: 'https://',
                            min: 6,
                            required: true
                        },
                        {
                            type: "checkbox",
                            label: "Open in new window?",
                            model: "newWindow",
                            default: true
                        }
                    ]
                },
                linkFormOptions: {
                    validateAfterLoad: true,
                    validateAfterChanged: true
                }
            }
        },
        watch: {
            cardSize: function (newCardSize) {
                this.updatedIsotopeLayout('masonry')
            }
        },
        computed: {
            selectedPath(){
                return $perAdminApp.getNodeFromViewOrNull('/state/pagebrowser/selectedPath')
            },
            withLinkTab(){
                return $perAdminApp.getNodeFromViewOrNull('/state/pagebrowser/withLinkTab')
            },
            tabIndicatorPosition(){
                let position
                switch(this.tab) {
                    case ('browse'):
                        position = 0
                        break
                    case ('cards'):
                        position = 72
                        break
                    case ('link'):
                        position = 144
                        break
                    default:
                        position = 0
                        break
                }
                return position
            },
            searchTabOffset(){
                if(this.withLinkTab){
                    return 216
                } else {
                    return 144
                }
            },
            path() {
                let root = $perAdminApp.getNodeFromViewOrNull('/state/pagebrowser/root')
                return root
            },
            nodes() {
                let view = $perAdminApp.getView()
                let nodes = view.admin.pathBrowser
                if(nodes && this.path) {
                    let nodesFromPath = $perAdminApp.findNodeFromPath(nodes, this.path)
                    return nodesFromPath
                }
                return {}
            },
            list(){
                return this.nodes.children || []
            }
        },
        methods: {
            cardIconSize: function(cardSize){
                return Math.floor(cardSize/3)
            },
            getImagesLoadedCbs: function() {
              return {
                progress: (instance, img ) => {
                },
                always: (instance) => {
                },
                done: (instance) => {
                },
                fail: (instance) => {
                }
              }
            },
            updatedIsotopeLayout: function(layout){
                this.$refs.isotope.layout(layout)
            },
            getIsotopeOptions: function() {
                return {
                    layoutMode: 'masonry',
                    itemSelector: '.card',
                    stamp: '.stamp',
                    masonry: {
                        gutter: 15
                    },
                    getSortData: {
                        name: function(itemElem){
                            return itemElem.name.toLowerCase()
                        },
                        created: function(itemElem){
                            return Date.parse(itemElem.created)
                        }
                    }
                }
            },

            onSort(sortType){
                this.sortBy = sortType
                this.$refs.isotope.sort(sortType)
            },

            select(name) {
                this.tab = name
            },
            searchFilter(item) {
                if(this.search.length === 0) return true
                return (item.name.indexOf(this.search) >= 0)
            },
            selectParent() {
                let parentFolder = this.path.split('/')
                parentFolder.pop()
                let newPath = parentFolder.join('/')
                this.selectFolder({ path: newPath} )
            },
            display(item) {
                return item.name !== 'jcr:content'
            },
            isFolder(item) {
                const FOLDERS = [
                    'per:Page',
                    'nt:folder',
                    'sling:Folder',
                    'sling:OrderedFolder'
                ]
                return FOLDERS.indexOf(item.resourceType) >= 0
            },
            isSelected(path) {
                return this.selectedPath === path 
            },
            selectFolder(item) {
                $perAdminApp.getApi().populateNodesForBrowser(item.path, 'pathBrowser').then( () => {
                    let pb = $perAdminApp.getNodeFromView('/state/pagebrowser')
                    pb.root = item.path
                    if(this.tab === 'cards'){
                        this.updatedIsotopeLayout('masonry')
                    }
                    this.preview = item
                })
            },
            selectItem(path) {
                return $perAdminApp.getNodeFromView('/state/pagebrowser').selectedPath = path
            },
            onUnlink() {
                this.setItemPath('')
                this.onHide()
            },
            onHide() {
                return $('#pageBrowserModal').modal('close')
            }
        }
    }
</script>
