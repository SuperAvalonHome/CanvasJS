<template>
  <div class="dashboard">
    <template>
      <div class="nav">
          <div class="left">
            <p class="business-name">{{baseProjectData.businessname}} </p>
          </div>
          <div class="right">
          <el-button icon="el-icon-download" size="mini" @click="doClearCategory(true)" style="" title="更新分类图标" circle></el-button>
          <el-button icon="el-icon-refresh" size="mini" @click="resetScaleX()" style="" title="重置图例缩放" circle></el-button>
          <el-dropdown @command="handleUserCommand" style="margin: 0 10px 0 10px;">
            <span class="el-dropdown-link">
                <el-button icon="el-icon-user-solid" size="mini" circle></el-button>
            </span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item command="setting">设置</el-dropdown-item>
              <el-dropdown-item command="logout" title="退出登录">退出</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
          <el-radio-group v-model="proSetting.thumbModel" size="mini" @change="handleModelChange($event)" v-if="proSetting.createModel == 'drawing'">
            <el-radio-button label="mini">小图</el-radio-button>
            <el-radio-button label="small">中图</el-radio-button>
            <el-radio-button label="medium">大图</el-radio-button>
          </el-radio-group>
          </div>
      </div>
    </template>

    <el-button class="left-drawer-open" icon="el-icon-right" size="mini" @click="leftDrawer = true" v-if="leftDrawerVisible" circle></el-button>

    <el-drawer
      :visible.sync="leftDrawer"
      size="230px"
      direction="ltr"
      >
      <div class="left-drawer-list">
      <draggable class="list-group" tag="ul" v-model="leftAlbums" v-bind="dragOptions" :move="leftAlbumOnMove" @start="isDragging=true" @end="isDragging=false">
        <transition-group type="transition" :name="'flip-list'">
          <div class="albums-box" v-for="(image,index) in leftAlbums" :key="image.name">
            <el-button class="remove-albums" icon="el-icon-delete" size="mini" @click="removeAlbums(image.name)" style="" title="移除图片" circle></el-button>
            <el-image class="left-drawer-img" fit="scale-down" :src="image.url" @click="leftAlbumClick(image.name)"></el-image>
          </div>
        </transition-group>
      </draggable>
      </div>
    </el-drawer>

    <div class="container">
      <section class="main-left" :style="'height:'+RightBarHeight+'px;'">
          <el-tabs v-model="activeNameLeft" type="card">
              <el-tab-pane label="房间(区域) &产品" name="skuProductTab" >
                <div class="left-floor-list">
                <el-tree
                  :data="projectData"
                  show-checkbox
                  node-key="id"
                  size="mini"
                  ref="productTree"
                  draggable
                  default-expand-all
                  :allow-drop="allowDrop"
                  :allow-drag="allowDrag"
                  @node-drop="doChangeProductTree"
                  @node-drag-start="doDragProductTree"
                  @check="doClickProductTree"
                  @node-click="doClickProductTree"
                  :expand-on-click-node="false">
                  <span class="custom-tree-node" slot-scope="{ node, data }">
                   <el-tooltip class="item" effect="light" :content="node.label" placement="right">
                    <span @click="changeLabel(node.data)" :ref="node.data.id" class="custom-tree-label">
                      <i title="点击添加到图纸" @click="showSkuIcon(node)" class="el-icon-warning" style="margin-right: 5px;" v-if="proSetting.createModel == 'drawing' && node.data.d_set == 'hide'"></i>{{ node.label }}</span>
                  </el-tooltip>
                  <span v-if="delNode(node)">
                  <el-button
                    type="text"
                    size="mini"
                    @click="() => remove(node, data)">
                    <i class="el-icon-delete el-icon--right"></i>
                  </el-button>
                  </span>
                </span>
                </el-tree>
                </div>
              </el-tab-pane>
              <el-tab-pane label="功能清单" name="tagProductList">
                <!--功能清单-->
                <div id="tagProductList">
                <el-card class="box-card">
                <div slot="header" class="clearfix">
                  <span style="font-size:14px;">功能清单</span>
                </div>
      			    <el-table
      			      :data="tagProductListFilter(1)"
      			      size="small"
      			      style="width: 100%">
      			      <el-table-column
      			        prop="name"
      			        label="名称"
      			        align="left"
      			        width="136px">
      			      </el-table-column>
      			      <el-table-column
      			        prop="num"
      			        label="数量"
      			        align="center"
      			        width="50px">
      			      </el-table-column>
      			    </el-table>
                </el-card>
                </div>
                <!--功能清单 end-->
                <!--产品清单-->
                <div id="tagSkuProductList">
                <el-card class="box-card" >
                <div slot="header" class="clearfix">
                  <span style="font-size:14px;">产品清单</span>
                </div>
                <el-table
                  :data="tagProductListFilter(2)"
                  size="small"
                  style="width: 100%">
                  <el-table-column
                    prop="name"
                    label="名称"
                    align="left"
                    width="136px">
                  </el-table-column>
                  <el-table-column
                    prop="num"
                    label="数量"
                    align="center"
                    width="50px">
                  </el-table-column>
                </el-table>
                </el-card>
                </div>
                <!--产品清单 end-->
              </el-tab-pane>
          </el-tabs>
      </section>
      <section class="main" :style="'width:'+CanvasWidth+'px;'+'height:'+FullScreenHeight+'px;'">
          <el-tabs v-model="mainTabsActive"  @tab-click="handleMainClick">
              <el-tab-pane label="点位图" name="canvasTab" v-if="proSetting.createModel == 'drawing'">
                <template>
                  <div id="canvasdemo" class="canvasdemo" :style="'min-width:'+CanvasWidth+'px;'+'min-height:'+CanvasHeight+'px;'">
                    <canvas id="canvid1" class="canvasmain" ></canvas>
                      <img :src="_src(icon)" :id="icon.id" v-for="icon in showIcons" :ref="icon.id" :key="icon.id" @load="loadImage(icon)" />
                  </div>
                  <el-slider
                  class="slider-item"
                    vertical
                    v-model="sliderScaleX"
                    @change="handleChangeScaleX"
                    :height="sliderHeight"
                    v-if="visibleSliderBar"
                    :step="10">
                  </el-slider>
                <div class="canvasButtons">
                </div>
                </template>
              </el-tab-pane>
              <el-tab-pane label="方案清单" name="skuListTab">
                <template>
                <div id="skuListDemo" class="skuListDemo" :style="'min-width:'+CanvasWidth+'px;'+'height:'+CanvasHeight+'px;'">
                <div class="main-project-list" v-for="(list,index) in computedSkuCateList">
                  <p class="main-project-list-title">{{list.label}}
                    <i class="el-icon-sort-down" @click="sortCateList(index, 'down')"></i>
                    <i class="el-icon-sort-up" @click="sortCateList(index, 'up')"></i>
                    <!--<el-checkbox :label="list.label" @change="handleCheckBoxChange($event,list.label)" :checked="CheckBoxChange(list.label)"></el-checkbox>-->
                    <label class="sub-title">合计：{{list.total_money}}</label>
                  </p>
                  <table>
                    <tbody>
                      <template v-for="(item,_index) in list.sku">
                        <tr><td>
                          <p class="list-label">{{item.label}}</p>
                          <div class="list-sort">
                            <i class="el-icon-sort-down" @click="sortSkuList(index, _index, 'down')"></i>
                            <i class="el-icon-sort-up" @click="sortSkuList(index, _index, 'up')"></i>
                            <!--<i class="el-icon-delete" @click="delSkuList(index, _index)" v-if="item.set_num == 0 || item.parent_id != 0"></i>-->
                          </div>

                        </td></tr>
                      <tr>
                        <td>
                          <div class="main-project-desc">
                            <img class="product-item-img" :src="skuImage(item.image)"/>
                            <p>
                              <span class="product-item-span">品牌：{{item.brand_name}}</span>
                              <span class="product-item-span">型号：{{item.model_name}}</span>
                              <span class="product-item-span">规格：<b class="item-value">{{item.item}}</b></span>
                            </p>
                          </div>

                        </td>
                        <td height="85px">
                         {{item.desc}}
                        </td>
                         <td>
                          <div class="item-price">
                            <p>单价：<el-input-number v-model="item.price" size="mini" controls-position="right" @change="(value) => handlePriceChange(value, item.id)" :min="lowPriceLimit(item)" :precision="2" :step="1"></el-input-number></p>
                            <p>数量：<el-input-number v-model="item.set_num" size="mini" controls-position="right" @change="(value) => handleNumChange(value, item.id)" :min="item.num" :max="9999"></el-input-number></p>
                          </div>
                        </td>
                         <td>
                          <div class="item-price">
                            <p>折扣：<el-input-number v-model="item.discount" size="mini" controls-position="right" @change="(value) => handleDiscountChange(value, item.id)" :min="item.mini_discount || 0" :max="100"></el-input-number> %</p>
                            <p>小计：<span class="item-value">{{item.total_money}}</span></p>
                          </div>
                        </td>
                      </tr>
                      </template>
                    </tbody>
                  </table>
                </div>
                </el-checkbox-group>

                <el-container class="setting-container" v-if="costWorking.type == 'normal'">
                  <el-main>
                  <el-row v-for="(costItem, i) in costWorking.normalCost" v-if="costItem.open" :key="i" class="setting-row">
                    <el-col :span="4"><el-input class="setting-input" v-model="costItem.item" size="mini" autocomplete="off" placeholder="自定义收费项目(8字内)"></el-input></el-col>
                    <el-col :span="8" v-if="costItem.class != 'total'" ><el-input class="setting-input" @change="costItemOpenChange" v-model="costItem.desc" size="mini" autocomplete="off" placeholder="自定义描述(20字内)"></el-input></el-col>
                    <el-col :span="3" v-if="costItem.class != 'total'" ><el-input class="setting-input" @change="costItemOpenChange" v-model="costItem.val" size="mini" label-width="6px" autocomplete="off"></el-input></el-col>
                    <el-col :span="3" v-if="costItem.class != 'total'" ><el-input class="setting-input" @change="costItemOpenChange" v-model="costItem.num" size="mini" label-width="6px" autocomplete="off"></el-input></el-col>
                    <el-col :span="3" v-if="costItem.class != 'total'" class="setting-span"><span>{{costItem.total}}</span></el-col>

                    <el-col :span="8" v-if="costItem.class == 'total'" >&nbsp;&nbsp;</el-col>
                    <el-col :span="3" v-if="costItem.class == 'total'" ><el-input class="setting-input" @change="costItemOpenChange" v-model="costItem.val" size="mini" label-width="6px" autocomplete="off"></el-input></el-col>
                    <el-col :span="3" v-if="costItem.class == 'total'" >&nbsp;&nbsp;</el-col>
                    <el-col :span="3" v-if="costItem.class == 'total'" class="setting-span"><span>{{costItem.total}}</span></el-col>

                    <el-col :span="2"><el-switch v-if="costItem.class != 'base' && costItem.class != 'total'" v-model="costItem.open" @change="costItemOpenChange"></el-switch></el-col>
                  </el-row>
                  </el-main>
                </el-container>
                <el-container class="category-cost-container" v-if="costWorking.type == 'category'">
                  <el-main>
                  <el-row class="category-cost-header">
                    <el-col :span="12">合计</el-col>
                    <el-col :span="3">设备总价</el-col>
                    <el-col :span="3">服务费用</el-col>
                    <el-col :span="3">税金</el-col>
                    <el-col :span="3">小计</el-col>
                  </el-row>
                  <el-row v-for="(item, i) in costWorking.categoryCost" :key="i" class="category-cost-row">
                    <el-col :span="12"><el-input class="setting-input" v-model="item.category_text" size="mini"></el-input></el-col>
                    <el-col :span="3" ><label>{{getClassPrice(item.costItem, 'base')}}</label></el-col>
                    <el-col :span="3" ><label>{{getClassPrice(item.costItem, 'common')}}</label></el-col>
                    <el-col :span="3" ><label>{{getClassPrice(item.costItem, 'tax')}}</label></el-col>
                    <el-col :span="3" ><label>{{getClassPrice(item.costItem, 'total')}}</label></el-col>
                  </el-row>
                  <el-row class="category-cost-footer">
                    <el-col :span="12">项目总概算</el-col>
                    <el-col :span="3">{{getClassPriceAll(costWorking.categoryCost, 'base')}}</el-col>
                    <el-col :span="3">{{getClassPriceAll(costWorking.categoryCost, 'common')}}</el-col>
                    <el-col :span="3">{{getClassPriceAll(costWorking.categoryCost, 'tax')}}</el-col>
                    <el-col :span="3">{{getClassPriceAll(costWorking.categoryCost, 'total')}}</el-col>
                  </el-row>
                  </el-main>
                </el-container>
                </div>
                </template>
              </el-tab-pane>
              <el-tab-pane label="方案整合（内测）" name="mergeTab" class="mergeTab">
                <template>
                  <div class="my-drawing-box" :style="'width:'+CanvasWidth+'px;'+'height:'+CanvasHeight+'px;'">
                    <div v-show="image.name == leftAlbumsEdit" v-for="(image,index) in leftAlbums" :key="'lc-'+image.name">
                      <div :class="'my-drawing-'+image.name"  :style="'width:'+image.width+'px;'+'height:'+image.height+'px;'+'min-width:'+image.width+'px;'+'min-height:'+image.height+'px;'"></div>
                    </div>
                  </div>
                </template>
              </el-tab-pane>
          </el-tabs>
      </section>
      <section class="main-right" :style="'height:'+RightBarHeight+'px;'">
        <el-tabs v-model="activeNameRight" type="card">
            <el-tab-pane label="房间" name="roomTab">
            <template>
    		      <el-select v-model="floorSelected" size="mini" @change="floorChange" style="width:50%; margin-top:10px;" placeholder="请选择楼层">
    		        <el-option
    		          v-for="item in floorSelect"
    		          :key="item.id"
    		          :label="item.label"
    		          :value="item.id">
    		        </el-option>
    		      </el-select>
    		      <el-button type="primary" icon="el-icon-edit" size="mini" @click="doSubmitFloorEdit">保存</el-button>
              <div style="margin:10px 15px 0 15px;">
                <el-input size="mini" placeholder="请输入房间名称" v-model="addFloorName" class="input-with-select">
                  <el-button slot="append" icon="el-icon-plus" @click="doSubmitFloorAdd"></el-button>
                </el-input>
              </div>
    		      <el-checkbox-group class="room-group" v-model="roomsSelected" @change="roomChange" style="text-align: left;">
    		        <el-checkbox class="text item" v-for='(item) in roomSelect' :label="item.id" :key="item.id">{{item.label}}</el-checkbox>
    		      </el-checkbox-group>
    		    </template>
            </el-tab-pane>
            <el-tab-pane label="功能选择" name="tagProductTab">
              <el-tree :data="tagProduct" :props="selectedTagProduct" @node-click="doClickTagProduct"></el-tree>
            </el-tab-pane>
            <el-tab-pane label="产品选择" name="skuProductTab">
              <div class="block">
                <span class="demonstration"></span>
              </div>

              <div class="select-group">
                <el-cascader
                  size="mini"
                  v-model="selectedCategory"
                  class="sku-select"
                  :options="categoryList"
                  :props="{ expandTrigger: 'click',multiple: false, checkStrictly: true }"
                  placeholder="请选择分类"
                  @change="doChangeSkuCategoryList()"
                  ref="refHandle"
                  clearable>
                </el-cascader>

                <el-select class="sku-select" clearable v-model="querySkuParams.bid" size="mini" @change="doChangeSkuBrandList" :disabled="selectBranddisabled" placeholder="选择品牌">
                  <el-option
                    v-for="item in brandSelected"
                    :key="item.pid"
                    :label="item.brandname"
                    :value="item.pid">
                  </el-option>
                </el-select>

                <el-col :span="20">
                <el-cascader
                  size="mini"
                  v-model="selectedModel"
                  class="sku-select"
                  :options="modelSelected"
                  placeholder="请选择规格"
                  :props="{ expandTrigger: 'click',multiple: false }"
                  @change="doChangeSkuModelList()"
                  collapse-tags
                  ref="selectedModelCascader"
                  clearable>
                </el-cascader>
                </el-col>
                <el-col :span="20">
                  <el-input v-model="querySkuParams.keywords" autocomplete="off" placeholder="关键词模糊搜索" size="mini"></el-input>
                </el-col>
                <el-button class="search-btn" type="primary" :loading="querySkuLoading" size="mini" @click="doQuerySkuProduct"  style="margin-top:10px;">查询产品</el-button>
              </div>
                  <div class="product">
            		<!--产品列表-->
                    <ul class="product-list" v-loading="querySkuImageLoading">
                      <li class="product-item" v-for="item in skuCategoryList" :key="item.pid">
                          <a :href="env.WEB_URL + `/goods/${item.pid}`" target="_blank" ><img class="product-item-img" :src="item.image"/></a>
                          <div class="product-item-content">
                            <span class="product-item-span">品牌：{{item.brand}}</span>
                            <el-tooltip class="item" effect="light" :content="item.name" placement="top-start">
                              <span class="product-item-span">名称：{{item.name}}</span>
                            </el-tooltip>
                            <el-tooltip class="item" effect="light" :content="item.model" placement="top-start">
                               <span class="product-item-span">型号：{{item.model}}</span>
                            </el-tooltip>
                            <el-button class="sku-button" v-if="item.sku.length == 1" type="primary" :value="item.sku[0].item" :label="item.sku[0].item" @click="doAddSkuProduct(item.sku[0], false, item)" size="mini">{{item.sku[0].item}}</el-button>

                            <span class="product-item-span" v-if="item.sku.length > 1">规格：
                              <el-popover
                              	  class="sku-popover"
                                    placement="right"
                                    width="400"
                                    trigger="click">
                    				        <el-button type="primary" class="sku-button" :value="sku.item" :label="sku.item" :key="sku.id" @click="doAddSkuProduct(sku, false, item)" v-for="sku in item.sku" size="mini">{{sku.item}}</el-button>
                                    <el-button slot="reference" trigger="click" class="el-popover-button" type="text">请选择</el-button>
                  			      </el-popover>
                            </span>
                            <span class="product-item-span" v-if="item.relation_item && item.relation_item.length">关联产品：
                               <el-popover
                              	  class="sku-popover"
                                    placement="right"
                                    width="400"
                                    trigger="click">
                                    <div v-for="relationItem in item.relation_item" class="relation-item" style="display:flex;align-items:center;">
                                      <a :href="env.WEB_URL_CN + `/maincorp/productdetail/${relationItem.pid}`" target="_blank" ><img class="product-item-img" :src="relationItem.image" style=" width:60px; height:60px; margin-right:10px;"/></a>
                                      <div class="relation-item-content" style="display:flex;flex-direction: column;justify-content: center;">
                                        <span class="relation-item-span">品牌：{{relationItem.brand}}</span>
                                        <span class="relation-item-span">名称：{{relationItem.name}}</span>
                                        <span class="relation-item-span">型号：{{relationItem.model}}</span>
                                        <span class="relation-item-span">规格：
                                           <el-button type="text" size="small" @click="doAddSkuProduct(sku, false, relationItem)" style="padding:0" :key="sku.id" v-for="sku in relationItem.sku">{{sku.item}}</el-button>
                                        </span>
                                      </div>
                                    </div>
                                    <el-button slot="reference" trigger="click" class="el-popover-button relation-button" type="text">请选择</el-button>
                  			      </el-popover>
                            </span>
                          </div>
                      </li>
                    </ul>
                    <div style="width:100%;">
                      <el-pagination
                        background
                        :hide-on-single-page="true"
                        layout="prev, pager, next"
                        :total="pagination.total"
                        :page-size="pagination.size"
                        :current-page.sync="pagination.pageNo"
                        :pager-count = 5
                      />
                    </div>
                  </div>
            </el-tab-pane>
        </el-tabs>
      </section>
    </div>
    <div class="footer canvasButtons">
      <template v-if="mainTabsActive == 'canvasTab'">
        <el-upload
          class="upload-demo btn-item"
          ref="upload"
          accept="image/jpeg,image/png"
          :file-list="uploadFile"
          :action="uploadUrl"
          :headers="respHeaders"
          :on-success="handleSuccess"
          :before-upload="beforeUpload"
          :show-file-list="false">
          <el-button size="mini" type="primary" ref="uploadBtn">上传<i class="el-icon-upload el-icon--right"></i></el-button>
        </el-upload>
        <el-button type="primary" size="mini" class="btn-item" id="savebutton" @click="doSaveImageCoords" >保存<i class="el-icon-success el-icon--right"></i></el-button>
        <el-button type="primary" size="mini" class="btn-item" @click="dialogCanvasPrint = true" >打印<i class="el-icon-printer el-icon--right"></i></el-button>
        <el-button type="primary" size="mini" class="btn-item" id="pngSkubutton3" @click="doSavePointList" >点位清单<i class="el-icon-download el-icon--right"></i></el-button>
      </template>
      <template v-if="mainTabsActive == 'skuListTab'">
        <el-upload
          v-if="proSetting.createModel == 'writing'"
          class="upload-demo btn-item"
          ref="upload"
          accept="image/jpeg,image/png"
          :file-list="uploadFile"
          :action="uploadUrl"
          :headers="respHeaders"
          :on-success="handleSuccess"
          :before-upload="beforeUpload"
          :show-file-list="false">
          <el-button size="mini" type="primary" ref="uploadBtn">上传<i class="el-icon-upload el-icon--right"></i></el-button>
        </el-upload>
        <el-button type="primary" size="mini" class="btn-item" id="saveSkubutton" @click="doSaveImageCoords" >保存<i class="el-icon-success el-icon--right"></i></el-button>
        <el-button type="primary" size="mini" class="btn-item" id="pngSkubutton5" @click="doDialogSkuProduct" >打印<i class="el-icon-printer el-icon--right"></i></el-button>
        <el-button type="primary" size="mini" class="btn-item" id="pngSkubutton2" @click="doSaveSkuCateProductList" >下载<i class="el-icon-download el-icon--right"></i></el-button>
        <!--<el-button type="primary" size="mini" class="btn-item" id="pngSkubutton6" @click="doDialogSkuProductLandscape" >横版打印<i class="el-icon-printer el-icon--right"></i></el-button>-->
        <el-popover
          placement="top-start"
          width="400"
          trigger="click">
          <template>
            <el-form class="costForm" label-width="140px">
            <el-form-item label="按分类计价">
              <el-switch
                v-model="costWorking.type"
                active-value="category"
                inactive-value="normal">
              </el-switch>
            </el-form-item>
            <el-form-item label="分类费用设置">
              <el-button size="mini" type="primary" icon="el-icon-edit":disabled="costWorking.type == 'normal'" @click="doOpenCategoryCost">设置</el-button>
            </el-form-item>
            <el-form-item label="产品单价选择">
              <el-select v-model="proSetting.usedPrice" size="mini" placeholder="请选择" @change="doChangePrice">
                <el-option
                  v-for="item in priceType"
                  :key="item.value"
                  :label="item.label"
                  :value="item.value">
                </el-option>
              </el-select>
            </el-form-item>
            <el-form-item label="项目备注">
              <el-input
                type="textarea"
                :rows="5"
                maxlength="200"
                placeholder="请输入内容"
                v-model="proSetting.proDescribe">
              </el-input>
            </el-form-item>
          </el-form>
          </template>
          <el-button slot="reference" size="mini" class="btn-item" type="primary">工程费用</el-button>
        </el-popover>
        <el-popover
          placement="top-start"
          title="小计"
          width="200"
          style="margin-left:8px;"
          trigger="click"
          v-if="copyRight.hasPermission == 1"
          :content="computedSkuCostPriceTotal">
          <el-button icon="el-icon-magic-stick" size="mini" circle slot="reference"></el-button>
        </el-popover>
        <el-popover
          placement="top-start"
          title="方案清单分享码(按功能)"
          width="300"
          popper-class="qrcode-popper"
          trigger="click">
          <img v-if="cateSkuListQR" :src="cateSkuListQR" alt="方案清单分享码"/>
          <el-button icon="el-icon-share" size="mini" circle slot="reference"></el-button>
        </el-popover>
        <el-popover
          placement="top-start"
          title="方案清单分享码(按房间)"
          width="300"
          popper-class="qrcode-popper"
          trigger="click"
          >
          <img v-if="roomSkuListQR" :src="roomSkuListQR" alt="方案清单分享码"/>
          <el-button icon="el-icon-share" size="mini" circle slot="reference"></el-button>
        </el-popover>
      </template>

      <template v-if="mainTabsActive == 'mergeTab'">

        <!--<el-button icon="el-icon-rank" size="mini" id="tool-select-shape" circle></el-button>-->
        <!--<el-button icon="el-icon-delete" size="mini" @click="leftAlbumRemove" circle></el-button>-->
        <el-divider direction="vertical"></el-divider>
        <el-button type="primary" size="mini" class="btn-item" id="mergebtn6" @click="openUserAlbums()">插入图片<i class="el-icon-circle-plus-outline el-icon--right"></i></el-button>
        <el-button type="primary" size="mini" class="btn-item" id="mergebtn2" @click="doDialogSkuProductLandscape()">插入清单<i class="el-icon-circle-plus-outline el-icon--right"></i></el-button>
        <el-button type="primary" size="mini" class="btn-item" id="mergebtn3" @click="doDialogCanvasPrint()">插入点位图<i class="el-icon-circle-plus-outline el-icon--right"></i></el-button>
        <el-button type="primary" size="mini" class="btn-item" id="saveSkubutton" @click="leftAlbumSave()" >保存<i class="el-icon-success el-icon--right"></i></el-button>
        <el-button type="primary" size="mini" class="btn-item" id="mergebtn4" @click="leftAlbumExport()">打印<i class="el-icon-printer el-icon--right"></i></el-button>
        <!--<el-button type="primary" size="mini" class="btn-item" id="mergebtn5" @click="leftAlbumTest()">分享<i class="el-icon-share el-icon--right"></i></el-button>-->

        <el-popover
          placement="top-start"
          title="方案分享"
          width="300"
          popper-class="qrcode-popper"
          trigger="click">
          <img v-if="albumsQR" :src="albumsQR" alt="方案分享码"/>
          <el-button icon="el-icon-share" size="mini" circle slot="reference"></el-button>
        </el-popover>
      </template>
    </div>

  <el-dialog
    title="请选择制图模式"
    :show-close="false"
    :fullscreen="false"
    :close-on-click-modal="false"
    :close-on-press-escape="false"
    :visible.sync="configureModelVisible"
    width="30%">
    <template>
    <div>
      <div style="display: flex; margin-top: 20px; height: 100px; justify-content:center;">
        <transition name="el-zoom-in-center">
          <div v-show="configureModelBtnVisible" class="transition-box" @click="configureModelSubmit('writing')">制作清单</div>
        </transition>

        <transition name="el-zoom-in-center">
          <div v-show="configureModelBtnVisible" class="transition-box" @click="configureModelSubmit('drawing')">制作图纸+清单</div>
        </transition>
      </div>
    </div>
  </template>
  </el-dialog>

  <el-dialog title="分类收费设置" :visible.sync="costCategoryVisible">
    <!--<el-select size="mini" v-model="costCategorySelected" multiple placeholder="请选择">
      <el-option
        v-for="(item, i) in computedSkuCateLable"
        :key="i"
        :label="item"
        :value="item">
      </el-option>
    </el-select>
    <el-button type="primary" size="mini" @click="doAddCategoryCost()">新增</el-button>-->
      <el-collapse>
        <el-collapse-item :name="categoryCost.category_text" v-for="(categoryCost, i) in costWorking.categoryCost" :key="i">
        <template slot="title">
          {{implode(categoryCost.category)}} <!--<i class="header-icon el-icon-remove-outline" @click="deleteCategoryCost(i)"></i>-->
        </template>
        <div>
          <!-- <P>{{categoryCost.costItem}}</P> -->
          <el-row v-for="(costItem, j) in categoryCost.costItem" :key="j" class="setting-row">
            <el-col :span="4">
              <el-input class="setting-input" v-model="costItem.item" size="mini" autocomplete="off" :readonly="costItem.class == 'tax'" placeholder="自定义收费项目(8字内)"></el-input>
             <!--  <span>{{costItem}}</span> -->
            </el-col>
            <el-col :span="8"><el-input class="setting-input" v-model="costItem.desc" size="mini" autocomplete="off" placeholder="自定义描述(20字内)"></el-input></el-col>
            <el-col :span="3" v-if="costItem.type == 'const'"><el-input @change="updateCategoryCostEvent" class="setting-input" v-model="costItem.val" size="mini" label-width="6px" placeholder="请输入单价" autocomplete="off" type="Number"></el-input></el-col>
            <el-col :span="3" v-if="costItem.type == 'const'"><el-input @change="updateCategoryCostEvent" class="setting-input" v-model="costItem.num" size="mini" label-width="6px" placeholder="请输入数量" autocomplete="off" type="Number"></el-input></el-col>
            <el-col :span="6" v-if="costItem.type == 'ratio'"><el-input @change="updateCategoryCostEvent" class="setting-input" v-model="costItem.val" size="mini" label-width="6px" placeholder="请输入比例" autocomplete="off" @blur="completetext(costItem)"></el-input></el-col>
            <el-col :span="4"><span class="setting-span">{{costItem.total}}</span></el-col>
            <el-col :span="2"><el-switch @change="updateCategoryCostEvent" v-if="costItem.class != 'base' && costItem.class != 'total'" v-model="costItem.open"></el-switch></el-col>
          </el-row>
        </div>
        </el-collapse-item>
      </el-collapse>

    <div slot="footer" class="dialog-footer">
      <el-button size="mini" @click="costCategoryVisible = false">取 消</el-button>
      <el-button size="mini" type="primary" @click="doSaveCostWorking">保 存</el-button>
    </div>
  </el-dialog>

  <el-dialog title="收费设置" :visible.sync="settingFormVisible">
    <el-form>
      <el-container class="setting-container">
        <el-header>
        <el-row class="setting-header">
          <el-col :span="4">收费项目</el-col>
          <el-col :span="8">收费说明</el-col>
          <el-col :span="3">计费类型</el-col>
          <el-col :span="3">比例/单价</el-col>
          <el-col :span="3">数量</el-col>
          <el-col :span="3">是否启用</el-col>
        </el-row>
        </el-header>
        <el-main>
        <el-row v-for="(costItem, i) in costSetting" :key="i" v-if="costItem.editable" class="setting-row">
          <el-col :span="4">
            <el-input :readonly="costItem.class == 'tax'" class="setting-input" v-model="costItem.item" size="mini" autocomplete="off" placeholder="自定义收费项目(8字内)"></el-input>
          </el-col>
          <el-col :span="8"><el-input class="setting-input" v-model="costItem.desc" size="mini" autocomplete="off" placeholder="自定义描述(20字内)"></el-input></el-col>
          <el-col :span="4"><el-switch size="mini"
            @change="changeCostType(costItem)"
            :disabled="costItem.class == 'tax'"
            v-model="costItem.type"
            active-text="百分比"
            active-value="ratio"
            inactive-value="const"
            inactive-text="按单价">
          </el-switch></el-col>
          <el-col :span="6" v-if="costItem.type == 'ratio'"><el-input placeholder="请输入比例" class="setting-input" v-model="costItem.val" size="mini" label-width="6px" autocomplete="on" @blur="completetext(costItem)"></el-input></el-col>
          <el-col :span="3" v-if="costItem.type == 'const'"><el-input placeholder="请输入单价" class="setting-input" v-model="costItem.val" size="mini" label-width="6px" autocomplete="off" type="Number"></el-input></el-col>
          <el-col :span="3" v-if="costItem.type == 'const'"><el-input placeholder="请输入数量" class="setting-input" v-model="costItem.num" size="mini" label-width="6px" autocomplete="off" type="Number"></el-input></el-col>
          <el-col :span="2"><el-switch v-if="costItem.class != 'base' && costItem.class != 'total'" v-model="costItem.open"></el-switch></el-col>
        </el-row>
        </el-main>
      </el-container>
    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-checkbox v-model="updateCostSetting">保存并更新当前项目</el-checkbox>
      <el-button @click="settingFormVisible = false">取 消</el-button>
      <el-button type="primary" @click="doSaveSetting">保 存</el-button>
    </div>
  </el-dialog>

  <el-dialog title="产品清单筛选" :visible.sync="dialogSkuChoose">
  <el-form :model="form">
  <el-checkbox :indeterminate="isSkuChooseIndeterminate" v-model="checkSkuChooseAll" @change="handleCheckAllChange">全选</el-checkbox>
  <div style="margin: 15px 0;"></div>
  <el-checkbox-group v-model="skuCateChoose" size="mini" @change="handleCheckedSkuCateChoose">
    <el-checkbox v-for="label in computedSkuCateChoose" :label="label" :key="label">{{label}}</el-checkbox>
  </el-checkbox-group>
  </el-form>
  <div slot="footer" class="dialog-footer">
    <el-button size="mini" @click="dialogSkuChoose = false">取 消</el-button>
    <el-button size="mini" @click="doSkuCateChoose" type="primary">确 定</el-button>
  </div>
</el-dialog>

  <el-dialog title="点位图打印" :visible.sync="dialogCanvasPrint">
  <el-form class="printForm" v-for="(printItem, i) in printForm" :key="i" label-width="80px">
    <el-form-item label="图纸名称" :label-width="formLabelWidth" >
      <el-input v-model="printItem.printDesc" autocomplete="off" placeholder="请输入图纸名称(8字内)" maxlength="8" size="mini" style="width:200px;float:left;"></el-input>
      <label class="el-form-item__label el-form-label" >分类图标</label>
      <el-switch size="mini"
        v-model="printItem.imageType"
        active-value="category"
        inactive-value="sku">
      </el-switch>
      <el-button type="danger" size="mini" icon="el-icon-delete" @click="printForm.splice(i, 1)"  circle></el-button>
    </el-form-item>
    <el-form-item label="选择打印功能图例" :label-width="formLabelWidth">
      <el-checkbox-group v-model="printItem.checkboxGroupTags" size="mini" style="float:left;">
        <el-checkbox :label="item.name" :value="item.name" :key="item.name" :checked="false" border v-for="item in currTagProductList"></el-checkbox>
      </el-checkbox-group>
    </el-form-item>
    <el-form-item label="选择打印产品图例" :label-width="formLabelWidth">
      <el-checkbox-group v-model="printItem.checkboxGroupSku" size="mini" style="float:left;">
        <el-checkbox :label="item.name" :value="item.name" :key="item.name" :checked="false" border v-for="item in currSkuProductList"></el-checkbox>
      </el-checkbox-group>
    </el-form-item>
    <el-divider></el-divider>
  </el-form>
  <div slot="footer" class="dialog-footer">
    <el-button size="mini" @click="addPrintForm" type="primary">新 增</el-button>
    <el-button size="mini" @click="printCanvas" type="primary">确 定</el-button>
    <el-button size="mini" @click="dialogCanvasPrint = false">取 消</el-button>
  </div>
</el-dialog>

  <el-dialog title="图库" :visible.sync="dialogUserAlbums">
    <div class="user-albums-dialog">
      <el-checkbox-group v-model="proSetting.usedAlbums">
      <div class="user-albums" v-for="(img, i) in userAlbums" :key="i">

        <el-button class="remove-albums" icon="el-icon-delete" size="mini" @click="removeUserAlbums(img)" style="" title="移除图片" circle></el-button>
        <el-image
          class="albums-img"
          :src="img"
          @click="clickUserAlbums(img)"
          fit="contain"></el-image>
          <el-checkbox
          :label="img"
          ></el-checkbox>
      </div>
      </el-checkbox-group>
    </div>
    <div slot="footer" class="dialog-footer">
      <div class="albums-btns">
        <el-upload
          ref="upload"
          accept="image/jpeg,image/png"
          :file-list="uploadFile"
          action="/Gateway/uploadUserAlbums/"
          :headers="respHeaders"
          :on-success="albumHandleSuccess"
          :before-upload="albumBeforeUpload"
          :show-file-list="false">
          <el-button size="mini" type="primary" ref="uploadBtn">上传<i class="el-icon-upload el-icon--right"></i></el-button>
        </el-upload>
        <el-button type="primary" size="mini" class="btn-item" @click="useAlbum()">插入图片<i class="el-icon-circle-plus-outline el-icon--right"></i></el-button>
      </div>
    </div>
  </el-dialog>

  <el-dialog title="点位图" :visible.sync="dialogPrintCanvas" id="dialogPrintCanvas" class="printCanvas-dialog" width="100%">
    <div class="page-list">
       <div class="btn-frame" style="margin-left:540px;">
        <span class="blk" @click="viewpdf('save', 'canvasPrint')" :class="{'is-disabled': !pdfC}"><i v-if="!pdfC" class="el-icon-loading"/>下载点位图</span>
        <span class="blk" @click="leftAlbumAddCanvasPrint()" :class="{'is-disabled': !pdfC}"><i v-if="!pdfC" class="el-icon-loading"/>插入点位图</span>
      </div>

      <div v-for="p in printCanvasData" name="print" class="printCanvasPage printCanvasHtml page landscape">
        <div class="wrap">
          <div class="print-container a4-broadwise" style="page-break-after:always">
          <div class="print-main">
            <div class="img-wrap">
              <img :src="p.base64" style="max-width:100%;max-height:100%;" />
            </div>
            <div class="info-wrap">
              <p class="top-info">
                  <img style="height:96%;" :src="copyRight.logoimage"/>
              </p>
              <p class="top-info">
                <span class="k">{{p.desc}}</span>
                <span class="f">{{p.floorLabel}}</span>
              </p>
              <div class="tag-list">
                <div class="tag-item" v-for="val in p.printSkuList">
                    <img :src="val.base64" />
                    <p class="item-info">
                        <span class="red" >x{{val.num}}</span>
                        <span> {{val.label}}</span>
                    </p>
                </div>
              </div>
            </div>
          </div>
          </div>
        </div>
      </div>
    </div>
  </el-dialog>

  <el-dialog title="产品清单" :visible.sync="dialogSkuProduct" id="productList" class="productList-dialog">
    <div class="page-list">
       <div class="btn-frame">
        <span @click="viewpdf('save', 'skuList')" :class="{'is-disabled': !pdf}"><i v-if="!pdf" class="el-icon-loading"/>下载清单</span>
        <span @click="viewpdf('preview', 'skuList')" :class="{'is-disabled': !pdf}"><i v-if="!pdf" class="el-icon-loading"/>预览清单</span>
        <!--<span @click="leftAlbumAddSkuList()" :class="{'is-disabled': !pdf}"><i v-if="!pdf" class="el-icon-loading"/>插入清单</span>-->
      </div>
      <div v-for="p in pages" name="print" class="skuListPage page rawPage">
        <div class="wrap">
          <div v-for="r in p" :class="r.className" v-html="r.innerHTML"></div>
        </div>
      </div>
    </div>
  </el-dialog>
  <div v-if="dialogSkuProduct">
  <div  class="hidden dom-render">
     <Header :d="copyRight" :c="companyInfo"/>
     <Footer :d="copyRight"/>
      <div v-for="(d,index) in computedSkuCateList" :key="d.label">
        <section>
          <div :class="[ index !== 0 && costWorking.type == 'category' ? 'next' : '','hd','row']">
            <div class="wrap">
              <p class="k p">{{d.label}} <label>合计：{{d.total_money}}</label></p>
            </div>
          </div>
          <div class="row bd" v-for="(item,_index) in d.sku" style="height:100%">
            <div class="wrap">
              <div class="b">
                <p class="item-label">{{item.label}}</p>
                <div class="item-content img">
                  <img class="item-img" :ref="'productImage' + item.id" :src="item.sku_img_m" :d="false" @load="loadImg(item.id, item.sku_img_m)"/>
                  <!--<img class="item-img" :ref="'productImage' + item.id" :src="item.image" :d="false" @load="loadImg(item.id, item.image)"/>-->
                    <p  class="item-info">
                      <span>品牌：{{item.brand_name}}</span>
                      <span>型号：{{item.model_name}}</span>
                      <span>规格：<b class="red">{{item.item}}</b></span>
                    </p>
                </div>
              </div>
              <div class="b second">
                <span class="text">{{item.desc}}</span>
              </div>
              <div class="b">
                <div class="more">
                <p>单价：<span class="red">{{discountPrice(item)}}</span></p>
                <p>数量：<span>{{item.set_num}}</span></p>
                <p>小计：<span class="red">{{item.total_money}}</span></p>
                </div>
              </div>
            </div>
          </div>
      </section>
      <section style="margin-top:20px;" class="bill-list">
        <div class="row bd small-bd" v-if="costWorking.type == 'category'">
          <div class="wrap">
            <p class="k">服务项目</p>
            <p class="k">服务说明</p>
            <p class="k">单价/百分比</p>
            <p class="k">数量</p>
            <p class="k">小计</p>
          </div>
        </div>
        <div class="row bd small-bd" v-for="(costItem, i) in d.cost" v-if="costItem.open" :key="i">
          <div class="wrap">
            <p class="b">{{costItem.item}}</p>
            <p class="b">{{costItem.desc}}</p>
            <p class="b">{{costItem.val}}</p>
            <p class="b">{{costItem.num}}</p>
            <p class="b">{{costItem.total}}</p>
          </div>
        </div>
      </section>
      </div>
      <section style="margin-top:20px;" class="bill-list" v-if="costWorking.type == 'normal'">
        <div class="row bd small-bd" v-for="(costItem, i) in costWorking.normalCost" v-if="costItem.open" :key="i">
          <div class="wrap">
            <p class="k">{{costItem.item}}</p>
            <p class="b">{{costItem.desc}}</p>
            <p class="b b-small">{{costItem.val}}</p>
            <p class="b b-small">{{costItem.num}}</p>
            <p class="b">{{costItem.total}}</p>
          </div>
        </div>
        <div class="row rd">
          <div class="wrap">
            <p class="desc">{{proSetting.proDescribe}}</p>
          </div>
        </div>
      </section>
      <section style="margin-top:20px;" class="bill-list" v-if="costWorking.type == 'category'">
        <div :class="[(computedSkuCateList && computedSkuCateList.length) ? 'next' : '','bd','row','small-bd']" >
          <div class="wrap">
            <p class="k">合计</p>
            <p class="k">设备总价</p>
            <p class="k">服务费用</p>
            <p class="k">税金</p>
            <p class="k">小计</p>
          </div>
        </div>
        <div class="row bd small-bd" v-for="(item, i) in costWorking.categoryCost" :key="i">
          <div class="wrap">
            <p class="b">{{item.category_text}}</p>
            <p class="b">{{getClassPrice(item.costItem, 'base')}}</p>
            <p class="b">{{getClassPrice(item.costItem, 'common')}}</p>
            <p class="b">{{getClassPrice(item.costItem, 'tax')}}</p>
            <p class="b">{{getClassPrice(item.costItem, 'total')}}</p>
          </div>
        </div>
        <div class="row bd small-bd">
          <div class="wrap">
            <p class="k">项目总概算</p>
            <p class="b">{{getClassPriceAll(costWorking.categoryCost, 'base')}}</p>
            <p class="b">{{getClassPriceAll(costWorking.categoryCost, 'common')}}</p>
            <p class="b">{{getClassPriceAll(costWorking.categoryCost, 'tax')}}</p>
            <p class="b">{{getClassPriceAll(costWorking.categoryCost, 'total')}}</p>
          </div>
        </div>
        <div class="row rd">
          <div class="wrap">
            <p class="desc">{{proSetting.proDescribe}}</p>
          </div>
        </div>
      </section>
  </div>
  </div>
  <PrintLandscape v-if="dialogSkuProductLandscape"
                  :visible.sync="dialogSkuProductLandscape"
                  :copyRight="copyRight"
                  :companyInfo="companyInfo"
                  :baseProjectData="baseProjectData"
                  :computedSkuCateList="computedSkuCateList"
                  :costWorking="costWorking"
                  :proSetting="proSetting"
                  @leftAlbumAddSkuList="leftAlbumAddSkuList"
  ></PrintLandscape>

  </div>
</template>

</template>


<script>
import Vue from 'vue'
import axios from "axios"
import jsPDF from 'jspdf'
import draggable from 'vuedraggable'
import Nav from "./Nav.vue";
import ProductList from "./ProductList.vue";
import PrintLandscape from "./PrintLandscape.vue";
import Header from "./Header.vue";
import Footer from "./Footer.vue";
import Cover from "./Cover.vue";
import openApi from "@/api/openApi.js"
import dashboardApi from "@/api/dashboard.js"
import {getUrlParams, getUUID, closeWindow, getURLBase64, toggleClass} from "@/js/public.js"

var YD = YAHOO.util.Dom;
var YE = YAHOO.util.Event;

var app;

var CanvasDemo = function() {
  var canvas1;
  var img = [];
  //var isMousedown = false;
  //var mouseStarX = 0;
  //var mouseStarY = 0;

  return {
    //
    init: function() {
      let _width = 0;
      let _height = 0;

      app.showIcons.forEach(function(val, index, data){

       let img_index = img.findIndex(d => d._oElement.id === val.id);
        if (img_index == -1) {
          img[img.length] = new Canvas.Img(val.id, {});
        } else {
       //   console.log("init.app.Icons.forEach img重复", val.id, img_index)
        }

        if (index == 0) {
            _width = img[0]['currentWidth'];
            _height = img[0]['currentHeight'];
            let x = 250 + parseInt(_width / 2);
            let y = 90 + parseInt(_height / 2);

            if (!isNaN(x) && !isNaN(y) && x > 250 && y > 90) {
              //app.setinitPosition(x, y);
            }
        }
      });

      canvas1 = new Canvas.Element();
      canvas1.init('canvid1',  { width: _width, height: _height });
      canvas1.initExtEvents('onclick',  this.onclick);
      canvas1.initExtEvents('ondblclick',  this.ondblclick);

      img.forEach(function(val, index, data){
        if (index == 0) {
            canvas1.setCanvasBackground(val);
        } else {
            canvas1.addImage(val);
        }
      });

      app.showIcons.forEach(function(val, index, data){
        if (index > 0) {
            canvas1._aImages[index-1] && canvas1._aImages[index-1].setImageCoords(val['data']);
        }
      });

      this.initEvents();

      if (app.showIcons.length >= 1) {
        canvas1.renderAll();
      }
    },
    ondblclick: function(imageCoords) {
      let sku = app.stashSkuData;
      let item = app.stashItemData;
      if (sku) {
        app.stashImageCoords = imageCoords
        app.doAddSkuProduct(sku, true, item);
      }
    },
    onclick: function(mousePoint, oImg) {
      if (oImg === false) return;
      app.Icons.some((val,index,array)=>{
        if (val.id == oImg._oElement.id) {
          val.d_set = "hide"
          CanvasDemo.removeIcon(oImg._oElement.id);
          return true;
        }
      })
      app.upTreeNode(oImg._oElement.id, {d_set:"hide"})
    },
    iconCover: function(mousePoint, iconPoint, iconSize) {
        let leftTopPoint = {top: iconPoint.top - iconSize.height / 2, left: iconPoint.left - iconSize.width / 2}
        let leftBottomPoint = {top: iconPoint.top + iconSize.height / 2, left: iconPoint.left - iconSize.width / 2}
        let rightTopPoint = {top: iconPoint.top - iconSize.height / 2, left: iconPoint.left + iconSize.width / 2}
        let rightBottomPoint = {top: iconPoint.top + iconSize.height / 2, left: iconPoint.left + iconSize.width / 2}

        if (mousePoint.top >= leftTopPoint.top && mousePoint.top <= leftBottomPoint.top && mousePoint.left >= leftTopPoint.left && mousePoint.left <= rightTopPoint.left) {
            return true
        }
        return false
    },
    initEvents: function() {
      YE.on('togglebg','click', this.toggleBg, this, true);
      //YE.on('showcorners','click', this.showCorners, this, true);
      YE.on('togglenone','click', this.toggleNone, this, true);
      YE.on('toggleborders','click', this.toggleBorders, this, true);
      YE.on('togglepolaroid','click', this.togglePolaroid, this, true);
      YE.on('pngbutton','click', function() { this.convertTo('png') }, this, true);
      YE.on('savebutton','click', this.saveData, this, true);
      YE.on('canvasdemo', 'mousedown', this.mousedown, this, true);
      YE.on('canvasdemo', 'mousewheel', this.mousewheel, this, true);
      //YE.on('canvasdemo', 'mousemove', this.mousemove, this, true);
      //YE.on('canvasdemo', 'mouseup', this.mouseup, this, true);
    },
    mousedown: function(event) {
      app.mousePoint = event
      return
      isMousedown = true;
      mouseStarX = event.offsetX;
      mouseStarY = event.offsetY;
    },
    mousemove: function(event) {
      //console.log("mousemove", event)
      return
      const _this = this;
      if (isMousedown) {
          let mouseEndX = event.offsetX;
          let mouseStarAndEndX = mouseStarX - mouseEndX;
          if (mouseStarAndEndX !== 0) {
              let scrollLeft = document.querySelector("#canvasdemo").scrollLeft + mouseStarAndEndX;
              if (scrollLeft > 0) {
                  document.querySelector("#canvasdemo").scrollLeft = scrollLeft;
              }
           }

          let mouseEndY = event.offsetY;
          let mouseStarAndEndY = mouseStarY - mouseEndY;
          if (mouseStarAndEndY !== 0) {
              let scrollTop = document.querySelector("#canvasdemo").scrollTop + mouseStarAndEndY;
              if (scrollTop > 0) {
                  document.querySelector("#canvasdemo").scrollTop = scrollTop;
              }
           }
       }
    },
    mouseup: function() {
      isMousedown = false;
    },
    renderCanvas: function() {
      /*let iconSize = app.iconSize[app.proSetting.thumbModel]
      canvas1._aImages.map(function(val,index,array){
        if (index > 0) {
          canvas1._aImages[index].setNormalizedSize(iconSize);
        }
      })*/
      canvas1.renderAll(true);
    },
    realRenderCanvas: function(ids) {
      if (ids.length > 0) {
        ids.forEach(function(val,index,array){
          let canvas_index = canvas1._aImages.findIndex(d => d._oElement.id === id);
          if(canvas_index >= 0) {
            canvas1._aImages[canvas_index].selected = true;
          }
        })

        canvas1.renderAll(false);
      }
    },
    addIcon: function(id, imageCoords) {
        setTimeout(()=>{

           let img_index = img.findIndex(d => d._oElement.id === id);

           let canvas_index = -1;

           if (canvas1._aImages != null) {
                canvas_index = canvas1._aImages.findIndex(d => d._oElement.id === id);
           }

           let obj = new Canvas.Img(id, {"randomposition": true, "disableAngle": true, "scaleX": canvas1._scaleX});
           let _keys = Object.keys(imageCoords);

           try{
             if(canvas_index == -1){
               canvas1.addImage(obj);
               if (_keys.length > 0) {
                    let cs_index = canvas1._aImages.findIndex(d => d._oElement.id === id);
                    canvas1._aImages[cs_index].setImageCoords(imageCoords);
               }
             } else {
             //   console.log("addIcon canvas1._aImages 重复", id, canvas_index)
             }

            if (img_index== -1) {
              img[img.length] = obj;
            } else {
           //   console.log("addIcon img重复", id, img_index)
            }

            canvas1.renderAll();
           }catch(e){
              console.log(e)
           }
        }, 100);
    },
    debugIcon: function(){
      let tempc = Object.assign({}, img);
      console.log("debugIcon", tempc)
    },
    removeIcon: function(id) {
        let img_index = img.findIndex(d => d._oElement.id === id);
        if (img_index >= 0) {
          img.splice(img_index, 1);
        }

        if (typeof(canvas1) == "undefined" || canvas1._aImages === null || canvas1._aImages.length == 0) return;
        let canvas_index = canvas1._aImages.findIndex(d => d._oElement.id === id);

        if (canvas_index >= 0) {
          canvas1.removeImage(canvas_index);
        }
    },
    mousewheel: function(event) {
      //console.log(event)
      event.preventDefault();
      let delta = 0;
      if (event.wheelDelta) {
        delta = event.wheelDelta/120;
        if (window.opera) delta = -delta;
      } else if (event.detail) {
        delta = -event.detail/3;
      }
      if (delta) {
        this.setScaleX(delta)
      }
    },
    setScaleX: function(delta) {

      if (delta == 1) {
        if (canvas1._scaleX >= 1.5) {
          //app.showMsg("图纸已调整到最大尺寸", "warning")
          return
        }
        canvas1._scaleX += 0.05;
      } else if (delta == -1) {
        if (canvas1._scaleX <= 0.1) {
          //app.showMsg("图纸已调整到最小尺寸", "warning")
          return
        }
        canvas1._scaleX -= 0.05;
      } else if (delta == 10000) {
        canvas1._scaleX = 1;
      } else {
        return
      }

      let scalex = canvas1._scaleX

      app.Icons.forEach(function(val, index, data){
        val['data']['scalex'] = scalex;
        val['data']['scaley'] = scalex;
        val['data']['hasScale'] = true;
        if (index == 0) {
            let scaleWidth = parseFloat(document.querySelector("#canvid1").width) * scalex;
            let scaleHeight = parseFloat(document.querySelector("#canvid1").height) * scalex;

            document.querySelector("#canvid1").style.width = scaleWidth + "px";
            document.querySelector("#canvid1").style.height = scaleHeight + "px";
            document.querySelector("#canvid1-canvas-background").style.width = scaleWidth + "px";
            document.querySelector("#canvid1-canvas-background").style.height = scaleHeight + "px";
            document.querySelector("#canvid1-canvas-container").style.width = scaleWidth + "px";
            document.querySelector("#canvid1-canvas-container").style.height = scaleHeight + "px";
            //document.querySelector("#canvasdemo").style.width = scaleWidth + "px";
            //document.querySelector("#canvasdemo").style.height = scaleHeight + "px";
        } else {
          let _id = canvas1._aImages.findIndex(d => d._oElement.id === val.id);
          if (_id >= 0) {
              canvas1._aImages[_id].setImageCoords(val['data']);
          }
        }
      });

      if (app.Icons.length > 1) {
          canvas1.renderAll(true);
      }
    },
    saveData: function() {
        if (typeof(canvas1) == 'undefined' || typeof(canvas1._aImages) == 'undefined' || canvas1._aImages == null) {
          return {'icons': app.Icons, 'tagProductList': app.tagProductList, 'skuList': app.skuProductList, 'skuCateSort': app.skuCateSort, 'skuDiscount': app.skuDiscount, 'checkedList': app.checkedList, 'categoryCost': app.categoryCost, 'proSetting': app.proSetting,'costWorking': app.costWorking, 'printForm': app.printForm}
        }

        for (var i = 0, l = canvas1._aImages.length; i < l; i += 1) {
            let _data = canvas1._aImages[i].getImageCoords();
            let _id = canvas1._aImages[i]._oElement.id;
            let _index = app.Icons.findIndex(d => d.id === _id);
            if(_index >= 0) {
              app.Icons[_index]['data'] = _data;
              app.Icons[_index]['load'] = 0;
            }
        }

        let cr_index = app.Icons.findIndex(d => d.id === 'Copyright-Icon');
        let ca_index = canvas1._aImages.findIndex(d => d._oElement.id === 'Copyright-Icon');
        if (cr_index >= 0) {
            app.Icons.splice(cr_index, 1);
        }
        if (ca_index > 0) {
            canvas1._aImages.splice(ca_index, 1);
        }

        let post = {'icons': app.Icons, 'tagProductList': app.tagProductList, 'skuList': app.skuProductList, 'skuCateSort': app.skuCateSort, 'skuDiscount': app.skuDiscount, 'checkedList': app.checkedList, 'categoryCost': app.categoryCost, 'proSetting': app.proSetting,'costWorking': app.costWorking, 'printForm': app.printForm};

        return post;
    },
    importData: function() {
      for (var i = 0, l = canvas1._aImages.length; i < l; i += 1) {
        for (let key in app.Icons){
          if (key == canvas1._aImages[i]._oElement.id){
            canvas1._aImages[i].setImageCoords(app.Icons[key]['data']);
            break;
          }
        }
      }
      canvas1.renderAll();
    },
    switchBg: function() {
      canvas1.fillBackground = (canvas1.fillBackground) ? false : true;
      canvas1.renderAll();
    },
    showCorners: function(id) {
      if (typeof(canvas1) == "undefined" || canvas1._aImages === null) return;
      for (var i = 0, l = canvas1._aImages.length; i < l; i += 1) {
        if (id == canvas1._aImages[i]._oElement.id){
          canvas1._aImages[i].setCornersVisibility(true);
        }else{
          canvas1._aImages[i].setCornersVisibility(false);
        }
      }
      canvas1.renderAll();
    },
    toggleNone: function() {
      if (canvas1._aImages === null) return;
      for (var i = 0, l = canvas1._aImages.length; i < l; i += 1) {
        canvas1._aImages[i].setBorderVisibility(false);
      }
      canvas1.renderAll();
    },
    toggleBorders: function() {
      if (canvas1._aImages === null) return;
      for (var i = 0, l = canvas1._aImages.length; i < l; i += 1) {
        canvas1._aImages[i].setBorderVisibility(true);
      }
      canvas1.renderAll();
    },
    togglePolaroid: function() {
      if (canvas1._aImages === null) return;
      for (var i = 0, l = canvas1._aImages.length; i < l; i += 1) {
        canvas1._aImages[i].setPolaroidVisibility(true);
      }
      canvas1.renderAll();
    },
    convertTo: function(format) {
      //var imgData = canvas1.canvasTo(format);
      //window.open(imgData, "_blank");
      app.printCanvas();
    },
    whatever: function(e, o) {
    }
  }
}();

export default {
  name: 'Dashboard',
  components: {
    Nav,
    ProductList,
    draggable,
    Header,
    Footer,
    Cover,
    PrintLandscape
    // BIllList
  },
  data () {
    return {
        //fixed
        PID: 0,
        FID: 0,

        editable: true,
        isDragging: false,
        delayedDragging: false,

        lc: null,
        LC: [],
        canvasBgColor: '#cccccc',
        primaryBgColor: '#000000',
        secondaryBgColor: '#ffffff',
        suo: 1,
        zoomBig: 0.1,
        zoomSmall: 0.1,
        moveX: 0,
        moveY: 0,
        moveSelfX: 0,
        moveSelfY: 0,
        inBoxWidth: 400,
        inBoxHeight: 400,
        inputNum: 5,
        zoomTimes: 3, // 滚轮滚动放大/缩小的倍数
        sizeIsNumber: true,
        jsonData: '',
        clientWidth: 0,

        leftDrawer:false,
        leftAlbumsEdit:'',
        userAlbums: [],
        leftAlbums: [
        //  {url:"http://localhost:8080/OpenApi/upload/albums/1.png", order:1, data: null, draw: [], name: 'test-1', width:900, height:600, fixed: false},
        //  {url:"http://localhost:8080/OpenApi/upload/albums/3.png", order:2, data: null, draw: [], name: 'test-2', width:900, height:600, fixed: false},
        ],
        imageBuffer: [],
        imageBufferC: [],
        imageBufferL: [],
        doc:{},
        docC:{},
        docL:{},
        pdf:"",
        pdfC:"",
        pdfL:"",

        leave_rid: 0,
        TOKEN: '',
        liveFrequency: 60000,
        autoSaveFrequency: 300000,
        autoCommitSwitch: false,
        proSetting: {thumbModel:"small", clearCategory:true, createModel:"drawing", usedPrice:"sale_price", usedAlbums:[], proDescribe:''},  //mini:20*20  small:40*40  medium:80*80 制图模式【drawing：制作图纸+清单，writing：制作清单】
        priceType: [
          {"label":"零售价","value":"sale_price"},
          {"label":"批发价","value":"trade_price"},
          {"label":"供货价A","value":"channel_a"},
          {"label":"供货价B","value":"channel_b"},
          {"label":"供货价C","value":"channel_c"}
        ],
        form: {printDesc: ''},
        printForm: [{printDesc:'',imageType:'sku',checkboxGroupTags:[],checkboxGroupSku:[]}],
        formLabelWidth: '140px',
        respHeaders: {},
        loading: {},
        initFinish: false,
        Icons: [],
        iconLength: 0,
        LoadIcons: [],
        copyRight: {},
        companyInfo: [],
        baseProjectData: {},
        selectedCategory: [],
        projectData: [],
        tagProductList: [],
        skuCateProductList:[],
        skuProductList: [],
        tagProduct: [],
        selectedTagProduct: [],
        skuCategoryList: [],
        categoryList: [],
        productSelected:[],
        checkedList:[],
        hasCheckedList: false,
        updateCostSetting: true,
        categoryCost:{item:[], open:false},
        addFloorName:'',
        selectedModel: '',
        mousePoint:{clientX:0,clientY:0,screenX:0,screenY:0},
        querySkuParams:{fid:'',sid:'',tid:'',bid:'',mid:'',lable:'',page:1,keywords:''},
        currNode_id: 0,		//当前房间id
        currNode_pid: 0,	//当前楼层id
        floorSelect: [],	//楼层数据
        floorSelected: '',	//楼层选择
        roomsSelected: [],	//已选择房间
        roomSelect: [],		//房间数据
        categoryListProps: { multiple: true },
        fSkuCategroy: [],
        sSkuCategroy: [],
        tSkuCategroy: [],
        brandSelected: [],
        modelSelected: [],
        checkboxGroupTags: [],
        checkboxGroupSku: [],
        skuCateChoose: [],
        uploadFile: [],
        uploadUrl: '',
        uploadAlbumUrl: '',
        mergeImageUrl: '',
        FullScreenWidth:window.screen.availWidth - 520,
        FullScreenHeight:window.screen.availHeight - 120,
        CanvasWidth:window.screen.availWidth - 508,
        CanvasHeight:window.screen.availHeight - 200,
        LCanvasWidth:978,
        LCanvasHeight:650,
        LeftFloorBarHeight:window.screen.availHeight - 160,
        RightBarHeight:window.screen.availHeight - 160,
        sliderHeight: (window.screen.availHeight - 300) + "px",
        mainTabsActive:'canvasTab',//1:canvasTab  2:skuListTab  3:mergeTab
        activeNameLeft:'skuProductTab',
        activeNameRight:'roomTab',
        sliderScaleX: 100,
        dialogSkuChoose: false,
        dialogCanvasPrint: false,
        dialogFilterTagList: false,
        dialogSkuProduct: false,
        dialogUserAlbums:false,
        dialogSkuProductLandscape: false,
        visibleSliderBar: false,
        querySkuLoading: false,
        querySkuImageLoading: false,
        uploadBGImage: false,
        selectBranddisabled: false,
        isSkuChooseIndeterminate: true,
        doQuerySkuProductCache: true,
        checkSkuChooseAll: true,
        stashSkuData: null,
        stashItemData: null,
        skuCateSort: {parent:[], children: []},
        iconSize: {mini:{height:20, width:20},small:{height:40, width:40},medium:{height:80, width:80}},
        stashImageCoords: {angle:0, left:0, top:0, scalex:1, scaley:1, theta:0},
        skuDiscount: {device:100, const:0, design:0, categoryCost:0, pretax:0, tax:16, total:100, items:{const_tips: '辅材及施工', design_tips: '设计与安装调试费'}},
        pages:[],
        landscapePages:[],
        skuProductListEmpty: false,
        pagination: {total: 0,size: 15,pageNo: 1},
        const_tips_editable:false,
        design_tips_editable: false,
        cateSkuListQR: '',
        roomSkuListQR: '',
        albumsQR: '',
        configureModelVisible: false,
        configureModelBtnVisible: true,
        settingFormVisible: false,
        costCategoryVisible: false,
        costSetting:[],
        costCategorySelected:[],
        costWorking:{type:'normal', 'normalCost': [], 'categoryCost': []},
        operation:'',
        dialogPrintCanvas:false,
        printCanvasData:[],
        initedLC:false,
        leftDrawerVisible:false,
    }
  },
  methods:{
    _src(icon){
      if (icon.model == 'mini' && icon.src_i) {
        return icon.src_i
      }else if (icon.model == 'medium' && icon.src_m) {
        return icon.src_m
      }else {
        //return icon.src + "?_r=" + Math.random()
        return icon.src
      }
    },
    async handleMainClick(element){
      if (element.name == "skuListTab") {
        if (this.cateSkuListQR.length == 0) {
          this.getQRCode('cateSkuListQR');
        }
        if (this.roomSkuListQR.length == 0) {
          this.getQRCode('roomSkuListQR');
        }
      }else if (element.name == "mergeTab" && this.initedLC === false) {
        this.openLoading()
        const res = await dashboardApi.getAlbums({pid:this.PID, fid:this.FID, token: this.TOKEN})

        if (this.roomSkuListQR.length == 0) {
          this.getQRCode('albumsQR');
        }

        this.loading.close()
        try {
          if (res.data.code == 200) {
            this.leftAlbums = res.data.data

            this.$nextTick(() => {
              this.initedLC = true
              let length = this.leftAlbums.length
              if (length > 0) {
                let last_name = this.leftAlbums[length - 1].name
                this.leftAlbumClick(last_name)
              }
            });

          } else {
            console.log(res.data.msg)
          }
        } catch (e) {
          console.log(e)
        }
      }

      if (element.name == "mergeTab") {
        this.leftDrawerVisible = true
      } else {
        this.leftDrawerVisible = false
      }
    },
    _src2 (url) {
      let _url = url.replace('/s_', '/m_');
      return _url
    },
    loadImg(id, url) {
      let _url = this._src2(url)
      let _ele = this.$refs['productImage' + id][0]
      if (_ele.getAttribute("d") == "true") {
        return true;
      }
      let img = getURLBase64(_url).then((result) => {
        _ele.setAttribute("src", result)
        _ele.setAttribute("d", "true")
      })
    },
    async base64Img(id, url) {
      let ele = this.$refs['productImage' + id][0]
      let img = getURLBase64(_src2(url)).then((result) => {
        ele.setAttribute("src", result)
      })
    },
    discountPrice(item) {
      return parseInt(item.price * item.discount / 100)
    },
    lowPriceLimit(item){
      if (this.proSetting.usedPrice == 'sale_price' && typeof(item.low_price) != "undefined") {
        return parseInt(item.low_price)
      } else {
         return 0
      }
    },
    changeCostType(item) {
      console.log("___",item)
      item.val = ''
    },
    completetext(item){
      if (item.val != '' && !/%$/.test(item.val)) {
        item.val = item.val + '%'
      }
      return true
    },
    implode(array){
      let str = ''
      array.forEach((val,index,arr) => {
        str += val + " & "
      })
      return str.substring(0, str.length-3)
    },
    getClassPrice(array, cate){
      let total = 0
      if (cate == 'common') {
        array.forEach((val,index,arr) => {
          if (val.open && val.class == cate && val.total != "" && !isNaN(val.total)) {
            total += parseFloat(val.total)
          }
        })
        return parseFloat(total).toFixed(2)
      } else {
        let i = array.findIndex(d => d.class == cate)
        if (i >= 0 && array[i].open) {
          total += parseFloat(array[i].total)
        }
        return total
      }
    },
    getClassPriceAll(data, cate){
      let total = 0
      data.forEach((val,index,arr) => {
        if (cate == 'common') {
          val.costItem.forEach((_val,_index,_arr) => {
            if (_val.open && _val.class == cate && _val.total != "" && !isNaN(_val.total)) {
              total += parseFloat(_val.total)
            }
          })
        } else {
          let i = val.costItem.findIndex(d => d.class == cate)
          if (i >= 0 && val.costItem[i].open  && val.costItem[i].total != "" && !isNaN(val.costItem[i].total)) {
            total += parseFloat(val.costItem[i].total)
          }
        }
      })
      return parseFloat(total).toFixed(2)
    },
    resetScaleX() {
      CanvasDemo.setScaleX(10000)
    },
    showMsg: function(msg, type) {
        this.$message({
          type: type,
          message: msg
        });
    },
    doAddCategoryCost: function() {
      let defaultCost = JSON.parse(JSON.stringify(this.costSetting))

      defaultCost[0].item = '设备小计'
      defaultCost[defaultCost.length-1].item = '小计'

      this.costWorking.categoryCost.push({
        'category': this.costCategorySelected,
        'category_text': this.implode(this.costCategorySelected),
        'category_real_text': this.implode(this.costCategorySelected),
        'costItem': defaultCost})

      this.costCategorySelected = []

      this.doUpdateCategoryCost()
      //this.computedSkuCateLable = []
    },
    deleteCategoryCost: function(i) {
      this.costWorking.categoryCost.splice(i, 1)
    },
    sortCateList: function(i, action) {
        let label_a = this.computedSkuCateList[i].label
        let sort_i = this.skuCateSort.parent.findIndex(i => i == label_a)
      if (action == 'up' && i > 0) {
        let label_b = this.computedSkuCateList[i-1].label
        let sort_j = this.skuCateSort.parent.findIndex(i => i == label_b)
        if (sort_i >= 0 && sort_j >= 0) {
          this.skuCateSort.parent[sort_j] = this.skuCateSort.parent.splice(sort_i, 1, this.skuCateSort.parent[sort_j])[0];
        }
      }else if (action == 'down' && i < this.computedSkuCateList.length - 1) {
        let label_c = this.computedSkuCateList[i+1].label
        let sort_k = this.skuCateSort.parent.findIndex(i => i == label_c)
        if (sort_i >= 0 && sort_k >= 0) {
          this.skuCateSort.parent[sort_k] = this.skuCateSort.parent.splice(sort_i, 1, this.skuCateSort.parent[sort_k])[0];
        }
      }
    },
    delSkuList: function(parent_i, i) {
      this.skuProductList.splice(i, 1)
    },
    sortSkuList: function(parent_i, i, action) {
        let parent_node = this.computedSkuCateList[parent_i]
        let parent_label = parent_node.label
        let id_i = parent_node.sku[i].id
        let sort_p = this.skuCateSort.children.findIndex(i => i.label == parent_label)
        let sort_i = this.skuCateSort.children[sort_p].sku.findIndex(i => i == id_i)
        if (action == 'up' && i > 0) {
          let id_j = parent_node.sku[i-1].id
          let sort_j = this.skuCateSort.children[sort_p].sku.findIndex(i => i == id_j)
          if (sort_i >= 0 && sort_j >= 0) {
            this.skuCateSort.children[sort_p].sku[sort_j] = this.skuCateSort.children[sort_p].sku.splice(sort_i, 1, this.skuCateSort.children[sort_p].sku[sort_j])[0];
          }
        }else if (action == 'down' && i < this.computedSkuCateList[parent_i].sku.length - 1) {
          let id_k = parent_node.sku[i+1].id
          let sort_k = this.skuCateSort.children[sort_p].sku.findIndex(i => i == id_k)
          if (sort_i >= 0 && sort_k >= 0) {
            this.skuCateSort.children[sort_p].sku[sort_k] = this.skuCateSort.children[sort_p].sku.splice(sort_i, 1, this.skuCateSort.children[sort_p].sku[sort_k])[0];
          }
        }
    },
    setinitPosition(x, y){
      this.stashImageCoords.left = x
      this.stashImageCoords.top = y
    },
    handleModelChange(e){
      return
      let label = ''
      if (this.proSetting.thumbModel == 'mini') {
        label = '小图'
      }else if (this.proSetting.thumbModel == 'small') {
        label = '中图'
      }else if (this.proSetting.thumbModel == 'medium') {
        label = '大图'
      }
     this.$confirm('确定要切换到'+label+'模式?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.doSaveImageCoords()
        this.redirect(this.FID)
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消切换'
        });
      });
    },
    CheckBoxChange(label){
      return this.checkedList.indexOf(label) > -1 ? true : false
    },
    handleCheckBoxChange(event,label){
      let _i = this.checkedList.indexOf(label)
      event ? this.checkedListPush(label) : this.checkedList.splice(_i, 1)
    },
    checkedListPush(label){
      let _i = this.checkedList.indexOf(label)
      if (_i == -1) this.checkedList.push(label)
    },
      skuImage (url) {
        let _url = url.replace('/s_', '/m_');
        return _url
      },
    addPrintForm() {
      let node = {printDesc:'',checkboxGroupTags:[],checkboxGroupSku:[]}
      this.printForm.push(node)
    },
      printSkuList(){
        var printHtml = document.getElementById("productList").innerHTML;
        var wind = window.open("",'newwindow', 'height=500, width=700, top=100, left=100, toolbar=no, menubar=no, scrollbars=no, resizable=no,location=n o, status=no');
        wind.document.body.innerHTML = printHtml;
        wind.print();
      },
      async printCanvas() {
        this.dialogCanvasPrint = false
        this.openLoading();

        let skulen = this.currSkuProductList.length
        const res = await dashboardApi.doMergeImage({pid:this.PID, fid:this.FID, token: this.TOKEN}, this.printForm)

        try {
          if (res.data.code == 200) {
            this.loading.close();
            let redirect = false
            if (redirect) {
              this.redirectPrint(res.data.data.cache_key);
            } else {
              this.printCanvasData = res.data.data.workData
              this.initPDFCanvas()
              this.dialogPrintCanvas = true
            }
          } else {
            console.log(res.data.msg)
          }
        } catch (e) {
          console.log(e)
        }

        this.loading.close();
      },
      async doSaveSkuCateProductList() {
        this.openLoading();
        let skuPriceList = this.getSkuPriceList()
        let postData = {skuPriceList: skuPriceList, computedSkuCateList: this.computedSkuCateList, costWorking:this.costWorking}
        const res = await dashboardApi.doSaveSkuCateProductList({pid:this.PID, fid:this.FID, token: this.TOKEN}, postData)
        try {
          if (res.data.code == 200) {
            this.loading.close();
            window.open("/Gateway/downProductListExcel/"+this.PID +"?token=" + this.TOKEN);
          } else {
            console.log(res.data.msg)
          }
        } catch (e) {
          console.log(e)
        }

        this.loading.close();
      },
      async doSaveSkuPriceList() {
        this.openLoading();
        let skuPriceList = this.getSkuPriceList()
        const res = await dashboardApi.doSaveSkuPriceList({pid:this.PID, fid:this.FID, token: this.TOKEN}, this.skuPriceList)
        try {
          if (res.data.code == 200) {
            this.loading.close();
            this.downloadSkuCateProductList()
          } else {
            console.log(res.data.msg)
          }
        } catch (e) {
          console.log(e)
        }

        this.loading.close();
      },
      doSavePointList () {
        window.open("/Gateway/downloadPointList/"+this.PID +"?token=" + this.TOKEN);
      },
      printCanvasBack2(dataURL){

        this.openLoading();

        try {
          let bytes = window.atob(dataURL.split(',')[1]);
          var ab = new ArrayBuffer(bytes.length);
          var ia = new Uint8Array(ab);
          for(var i = 0; i < bytes.length; i++){
              ia[i] = bytes.charCodeAt(i);
          }
          var blob = new Blob([ab], {type: 'image/png'});

          let formData = new FormData()
          let headers = {headers: {"Content-Type": "multipart/form-data", "Authorization": this.TOKEN}}

          formData.append('file', blob, this.PID + '-work.png');

          axios.post(this.uploadUrl + '/work', formData, headers).then(res => {
            this.redirectPrint();
          }).catch(err => {
            this.loading.close();
          })
        } catch(e) {
          this.loading.close();
        }

        this.loading.close();
      },
      printCanvasBack(){
        CanvasDemo.renderCanvas();
        var canvas = document.getElementById("canvid1");

        canvas.toBlob(function(blob) {
          var wind = window.open("",'newwindow', 'height=500, width=700, top=100, left=100, toolbar=no, menubar=no, scrollbars=no, resizable=no,location=n o, status=no');
          var newImg = wind.document.createElement("img");
          var url = URL.createObjectURL(blob);

          newImg.onload = function() {
            URL.revokeObjectURL(url);
          };

          newImg.src = url;
          wind.document.body.appendChild(newImg);
          wind.print();
        });
      },
      doFilterTagList(){
        let ids = []
        this.checkboxGroupTags.forEach(function(val,index,array){
          app.Icons.forEach(function(_val,_index,_array){
            if (_val.node.label == val) {
              ids.push(_val.id)
            }
          })
        })
        CanvasDemo.realRenderCanvas(ids);
      },
      doFilterSkuList(){
        let ids = []
        this.checkboxGroupSku.forEach(function(val,index,array){
          app.Icons.forEach(function(_val,_index,_array){
            if (_val.node.label == val) {
              ids.push(_val.id)
            }
          })
        })
        CanvasDemo.realRenderCanvas(ids);
      },
      handleChangeScaleX(val) {
          let scalex = val / 100;
          CanvasDemo.setScaleX(scalex);
      },
      albumBeforeUpload(File){

        let testmsg = File.name.substring(File.name.lastIndexOf('.')+1)
        let extension = testmsg.toLowerCase() === 'jpg'
        let extension2 = testmsg.toLowerCase() === 'png'
        let isLt2M = File.size / 1024 / 1024 < 2

        if(!extension && !extension2) {
          this.$message({
            message: '上传文件只接受 .jpg 或 .png 格式!',
            type: 'warning'
          });
          return false;
        }
        if(!isLt2M) {
          this.$message({
            message: '上传文件大小不能超过 2MB!',
            type: 'warning'
          });
          return false;
        }

        let that = this
        this.openLoading();
        try {
          let formData = new FormData()
          formData.append('file', File)
          let headers = {headers: {"Content-Type": "multipart/form-data", "Authorization": that.TOKEN}}
          axios.post(that.uploadAlbumUrl, formData, headers).then(res => {
            this.loading.close();
            if (res.data.code == 200) {

            } else {
              this.$message({
                message: '文件上传失败，请重新上传',
                type: 'warning'
              });
            }
          }).catch(err => {
            this.loading.close();
          })
        } catch (e) {
          this.loading.close();
        }

      },
      beforeUpload(File) {
        let testmsg = File.name.substring(File.name.lastIndexOf('.')+1)
        let extension = testmsg.toLowerCase() === 'jpg'
        let extension2 = testmsg.toLowerCase() === 'png'
        let isLt2M = File.size / 1024 / 1024 < 6

        if(!extension && !extension2) {
          this.$message({
            message: '上传文件只接受 .jpg 或 .png 格式!',
            type: 'warning'
          });
          return false;
        }
        if(!isLt2M) {
          this.$message({
            message: '上传文件大小不能超过 6MB!',
            type: 'warning'
          });
          return false;
        }

        if (!this.FID) {
          this.$message({
            message: '请先勾选左侧菜单栏的楼层',
            type: 'warning'
          });
          return false;
        }

        let that = this

        if (this.uploadBGImage) {
          let tips = ''
          if (this.proSetting.createModel == 'writing') {
            tips = '确定要上传图纸吗?'
          } else {
            tips = '确定要替换图纸吗?'
          }
           this.$confirm(tips, '提示', {
              confirmButtonText: '确定',
              cancelButtonText: '取消',
              type: 'warning'
            }).then(() => {
              this.openLoading();
              try {
                let formData = new FormData()
                let headers = {headers: {"Content-Type": "multipart/form-data", "Authorization": that.TOKEN}}

                formData.append('file', File)

                axios.post(that.uploadUrl + "/replace", formData, headers).then(res => {
                  if (res.data.code == 200) {
                    this.redirect(this.FID);
                  } else {
                    this.$message({
                      message: '文件上传失败，请重新上传',
                      type: 'warning'
                    });
                  }
                  //this.redirect(this.FID);
                  //that.handleSuccess(res.data);
                }).catch(err => {
                  this.loading.close();
                })
              } catch (e) {
                console.log(e)
                this.loading.close();
              }

            }).catch(() => {
              this.$message({
                type: 'info',
                message: '已取消上传'
              });
            });

            return false;
        }

        return true;
      },
      albumHandleSuccess(res) {
        if (res.code == 200) {
          this.getUserAlbums()
        } else {
          app.$message({
            message: '文件上传失败，请重新上传',
            type: 'warning'
          });
        }
      },
      handleSuccess(response) {
        if (response.code == 200) {
            this.uploadBGImage = true
            if (this.Icons && this.Icons.length > 0) {
              this.Icons = []
              //todo
              //this.tagProductList = []
              //this.skuProductList = []
              //this.skuCateProductList = []
            }

            this.Icons.push({
                id: response.data.id,
                src: response.data.src,
                op_index: 1,
                node: {
                    id: this.currNode_id,
                    pid: this.currNode_pid,
                    code: null,
                    label: null
                },
                data: {}
            });
            /*this.$nextTick(() => {
                setTimeout(()=>{
                  CanvasDemo.init();
                }, 2000);
            });*/
            this.$notify.close()
        } else {
          app.$message({
            message: '文件上传失败，请重新上传',
            type: 'warning'
          });
        }
      },
      doSkuCateChoose(data) {
        //console.log(data)
      },
      handleCheckAllChange(val) {
        this.skuCateChoose = val ? this.computedSkuCateChoose : [];
        this.isSkuChooseIndeterminate = false;
      },
      handleCheckedSkuCateChoose(value) {
        let checkedCount = value.length;
        this.checkSkuChooseAll = checkedCount === this.computedSkuCateChoose.length;
        this.isSkuChooseIndeterminate = checkedCount > 0 && checkedCount < this.computedSkuCateChoose.length;
      },
      async doQuerySkuProduct() {
        if (this.uploadBGImage === false && this.proSetting.createModel == "drawing") {
           this.$message({
                message: '请先上传当前编辑楼层的工程图',
                type: 'warning'
           });
          return;
        }

        this.querySkuLoading = true
        this.querySkuParams.page = this.pagination.pageNo
        //const res = await dashboardApi.doQueryProduct({token: this.TOKEN}, this.querySkuParams)
        const res = await openApi.doQueryProduct({token: this.TOKEN}, this.querySkuParams)

        try {
          this.querySkuLoading = false
          if (res.data.code == 200) {
            this.skuCategoryList = res.data.data.rows
            this.pagination.total = +res.data.data.total

            /*if (!this.querySkuParams.cache) {
              this.doClearCategory();
            }*/
          } else {
            console.log(res.data.msg)
          }
        } catch (e) {
          console.log(e)
        }
      },
      async doChangeSkuCategory(index = 1) {
          if(index == 1) {
              if(this.querySkuParams.fid) {
                let fCategroy = this.categoryList.find(d => d.value === this.querySkuParams.fid);
                this.sSkuCategroy = fCategroy.children;
              } else {
                this.sSkuCategroy = [];
                this.tSkuCategroy = [];
              }
              this.querySkuParams.sid = ""
              this.querySkuParams.tid = ""
              this.selectBranddisabled = false
          }else if(index == 2) {
              if (this.querySkuParams.sid) {
                let f_index = this.categoryList.findIndex(d => d.value === this.querySkuParams.fid);
                let sCategroy = this.categoryList[f_index].children.find(d => d.value === this.querySkuParams.sid);
                this.tSkuCategroy = sCategroy.children;
              } else {
                this.tSkuCategroy = [];
              }
              this.querySkuParams.tid = ""
              this.selectBranddisabled = false
          }else if(index == 3) {
              this.selectBranddisabled = true
              this.querySkuParams.bid = ""
              this.doQuerySkuProduct()
          }
      },
      async remove(node, data) {
        if (node.level == 1 || node.level == 2) {
          let _title = node.data.label;
          this.$confirm('确定要移除'+_title+'吗?', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(async () => {
            let fid = 0
            let rid = 0
            if (node.level == 2) {
              fid = node.data.pid
              rid = node.data.id
            } else if (node.level == 1) {
              fid = node.data.id
            }

            this.openLoading();

            let _data = CanvasDemo.saveData();
            let _res = await dashboardApi.doSaveImageCoords({pid:this.PID, fid: this.FID, token: this.TOKEN}, _data)

            const res = await dashboardApi.doDelete({pid:this.PID, fid:fid, rid:rid, token: this.TOKEN}, {})

            try {
              if (res.data.code == 200) {
                this.$message({
                  type: 'success',
                  message: '删除成功!'
                });
                if(fid == this.FID && rid == 0) {
                  this.redirect(0)
                } else {
                  this.redirect(this.FID)
                }
              } else {
                console.log(res.data.msg)
              }
              this.loading.close();
            } catch (e) {
              this.loading.close();
            }
          }).catch((e) => {
            this.$message({
              type: 'info',
              message: '已取消删除'
            });
          });
          return
        }

		    let _title = '';
      	if (data.type == 1) {
      		_title = '功能';
      	} else if (data.type == 2) {
      		_title = '产品';
      	}

        //console.log("remove start...")
        //CanvasDemo.debugIcon()

        this.$confirm('确定要移除该'+_title+'吗?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          let parent = node.parent;
          let children = parent.data.children || parent.data;
          let index = children.findIndex(d => d.id === data.id);
          let iconIndex = this.Icons.findIndex(d => d.id === data.icon_id);

	      	if (data.type == 1) {
            let _code = this.Icons[iconIndex]['node']['code'];
            let _label = this.Icons[iconIndex]['node']['label'];
            this.upTagProductList(_code, _label, 'remove', data.type);
	      	} else if (data.type == 2) {
            let _id = this.Icons[iconIndex]['node']['proid'];
            let _label = this.Icons[iconIndex]['node']['label'];
	      		this.upSkuProductList(_id, 'remove');
	      		this.upTagProductList(_id, _label, 'remove', data.type);
	      	}
          if (index >= 0) {
            children.splice(index, 1);
          }
          if (iconIndex >= 0) {
            this.Icons.splice(iconIndex, 1);
          }
          CanvasDemo.removeIcon(data.icon_id);

          this.$message({
            type: 'success',
            message: '删除成功!'
          });
        }).catch((e) => {
          console.log(e)
          this.$message({
            type: 'info',
            message: '已取消删除'
          });
        });
      },
      upSkuProductList: function(_id, action) {

        let that = this
        if (action == 'remove') {
          let fid = this.FID
          let remove = false

          this.skuProductList.some(function(val,index,array){
              if (val.id == _id) {
                  if (array[index].num > 1) {
                      array[index].num -= 1;
                      if (array[index].set_num) {
                        array[index].set_num -= 1;
                      } else {
                        array[index].set_num = array[index].num;
                      }

                      let discount = parseInt(array[index].discount);

                      let f_index = -1
                      if (typeof(array[index].fids) != "undefined" && (array[index].fids instanceof Array)) {
                        f_index = array[index].fids.findIndex(d => d.fid === fid);
                      } else {
                        f_index = -1
                      }

                      if (f_index >= 0) {
                        array[index].fids[f_index].num -= 1
                        let r_index = array[index].fids[f_index].rids.findIndex(i => i == that.currNode_id)
                        if (r_index >= 0) {
                          array[index].fids[f_index].rids.splice(r_index, 1)
                        }
                        //console.log("remove", f_index, r_index, that.currNode_id, array[index].fids[f_index].rids)
                        if(discount >= 0 && discount <= 10000) {
                          array[index].total_money = parseFloat(array[index].price * array[index].set_num * discount / 100).toFixed(2)
                        }else{
                          array[index].total_money = array[index].price * array[index].set_num
                        }
                      }
                  } else {
                      array.splice(index, 1);
                      remove = true
                  }
                  return true;
              }
          });

          if (_id) {
            for (let _i = 0; _i < this.skuProductList.length; _i++) {
              if (typeof(this.skuProductList[_i].parent_nums) != "undefined" && (this.skuProductList[_i].parent_nums instanceof Array)) {
                  let _j = this.skuProductList[_i].parent_nums.findIndex(d => d.pid == _id)
                  if (_j >= 0) {
                    let _num = this.skuProductList[_i].parent_nums[_j].num
                    if (this.skuProductList[_i].num > _num) {
                      this.skuProductList[_i].num -= _num
                      if (this.skuProductList[_i].set_num > _num) {
                        this.skuProductList[_i].set_num -= _num
                      } else {
                        this.skuProductList[_i].set_num = this.skuProductList[_i].num
                      }
                    } else {
                      this.skuProductList.splice(_i, 1)
                      _i--
                    }
                  }
                }else if (typeof(this.skuProductList[_i].parent_ids) != "undefined" && (this.skuProductList[_i].parent_ids instanceof Array)) {
                let _j = this.skuProductList[_i].parent_ids.findIndex(d => d == _id)
                if (_j >= 0) {
                  this.skuProductList[_i].parent_ids.splice(_j, 1);
                  if (this.skuProductList[_i].num > 1) {
                    this.skuProductList[_i].num -= 1
                    if (this.skuProductList[_i].set_num > 1) {
                      this.skuProductList[_i].set_num -= 1
                    } else {
                      this.skuProductList[_i].set_num = this.skuProductList[_i].num
                    }
                  } else {
                    this.skuProductList.splice(_i, 1)
                    _i--
                  }
                }
              }
            }
          }
        }
      },
      upTagProductList: function(_code, _label, action, _type ) {
        let fid = this.FID

        if (action == 'add') {
            let exist = false;
            this.tagProductList.some(function(val,index,array){
                if (val.name == _label && val.type == _type) {
                    array[index].num += 1;

                    let f_index = array[index].fids.findIndex(d => d.fid === fid);

                    if (f_index >= 0) {
                      array[index].fids[f_index].num += 1
                    } else {
                      array[index].fids.push({fid: fid, num: 1})
                    }

                    exist = true;
                    return true;
                }
            });
            if (exist === false) {
                let code = ICON[_code];
                this.tagProductList.push({
                    code: code,
                    name: _label,
                    type: _type,
                    fids: [{"fid": fid, "num":1}],
                    num: 1
                });
            }
        } else if (action == 'remove') {
            this.tagProductList.some(function(val,index,array){
                if (val.name == _label && val.type == _type) {
                    if (array[index].num > 1) {
                        let f_index = array[index].fids.findIndex(d => d.fid === fid);
                        if (f_index >= 0) {
                          array[index].fids[f_index].num -= 1
                        }
                        array[index].num -= 1;
                    } else {
                        array.splice(index, 1);
                    }
                    return true;
                }
            });
        }
      },
    doDragProductTree(before, leave) {
      this.leave_rid = parseInt(before.parent.data.id)
    },
    allowDrop(draggingNode, dropNode, type) {
      return dropNode.level === 2 && type == 'inner' && dropNode.parent.data.id == this.FID;
    },
    allowDrag(draggingNode) {
      return draggingNode.level === 3;
    },
    doChangeProductTree(before, after, inner, event) {
      if (after.level != 2) {
        return false
      }
      let sku_id = ''
      let icon_id = before.data.icon_id
      let _i = icon_id.indexOf('-')

      if (_i >= 0) {
        sku_id = icon_id.substring(0, _i)
      } else {
        sku_id = icon_id
      }

      let room_id = parseInt(after.data.id)
      let leave_rid = this.leave_rid

      if (leave_rid > 0 && leave_rid != room_id) {
        this.skuProductList.forEach((val,index,array)=>{
          if(val.id == sku_id){
            val.fids.map((_val,_index,_array)=>{
              if (_val.fid == this.FID) {
                  let f_index = _val.rids.findIndex(d => d == leave_rid)
                  val.fids[_index].rids[f_index] = room_id
                }
            })
          }
        })
        this.Icons.map(function(val,index,array){
          if (val.id == icon_id) {
            val.node.id = room_id
          }
        })
        this.leave_rid = 0
      }
    },
    doClickProductTree(data, node){

      if (node.level == 3 && typeof(data.icon_id) != "undefined") {
        CanvasDemo.showCorners(data.icon_id);
      }

        if (typeof(data.pid) != "undefined") {
            if (data.pid != this.currNode_pid) {
		        let currNode = this.getTreeNode(this.currNode_pid, 1)
                if (currNode.label) {
                    app.$confirm('确定要离开当前'+currNode.label+'楼层工作区吗?', '提示', {
                      confirmButtonText: '确定',
                      cancelButtonText: '取消',
                      type: 'warning'
                    }).then(() => {
                      if (this.proSetting.createModel == 'writing') {
                        dashboardApi.uploadDefault({pid:this.PID, fid:data.pid, token: this.TOKEN}, {})
                      }

                      this.doSaveImageCoords({}, false);
                      setTimeout(() => {
                        this.redirect(data.pid);
                      }, 1000);

                    }).catch(() => {
                        this.$refs.productTree.setCheckedKeys([this.currNode_id]);
                    });
                } else {
                  this.redirect(data.pid);
                }
            } else {
                this.currNode_id = parseInt(data.id);
            }
        } else {
            let strId = data.id.toString();
            this.currNode_id = strId.replace(/-\d+/, '');
        }

        this.currProjectListNode = data.id;
        this.$refs.productTree.setCheckedKeys([data.id]);
    },
	  delNode: function(node) {
	    if (typeof(node.data.del) != 'undefined') {
	        return true;
	    } else {
	        return true;
	    }
	  },
      roomChange: function(id) {
      },
      objectSpanMethod({ row, column, rowIndex, columnIndex }) {
        if (columnIndex === 0) {
          if (rowIndex % 3 === 0) {
            return {
              rowspan: 2,
              colspan: 1
            };
          } else {
            return {
              rowspan: 0,
              colspan: 0
            };
          }
        }
      },
      doAddSkuProduct(sku, stashModel = false, obj = {}, relation_part = []) {

        if (!this.currNode_id || this.currNode_id.toString().length < 5) {
           this.$message({
                message: '请正确勾选当前楼层和房间节点',
                type: 'warning'
           });
           return
        }

        this.stashSkuData = sku
        this.stashItemData = obj

        this.doClickSkuProduct(sku, stashModel, false, obj.name)

        if (typeof(obj.relation_part) != "undefined" && obj.relation_part.length > 0) {
          obj.relation_part.forEach(i => {
            this.doClickSkuProduct(i.sku[0], stashModel, sku.id, i.name, i.num)
            /*for (let j = 0; j < i.num; j++) {
              this.doClickSkuProduct(i.sku[0], stashModel, sku.id, i.name)
            }*/
          })
        } else if (relation_part.length > 0) {
          relation_part.forEach(i => {
            this.doClickSkuProduct(i.sku[0], stashModel, sku.id, i.name, i.num)
            /*for (let j = 0; j < i.num; j++) {
              this.doClickSkuProduct(i.sku[0], stashModel, sku.id, i.name)
            }*/
          })
        }
      },
      getSkuPriceList() {
        let pricelist = new Array();
        pricelist.push({item:"device", tips: "设备总价(不含税)", price: this.computedSkuPriceDevice, discount: this.skuDiscount.device})
        if (this.categoryCost.open) {
          pricelist.push({item:"categoryCost", tips: "产品分类工程费用总计", price: this.computedSkuCategoryCost, discount: this.skuDiscount.categoryCost})
        } else {
          pricelist.push({item:"design", tips: this.skuDiscount.items.design_tips, price: this.computedSkuPriceDesign, discount: this.skuDiscount.design})
        }
        pricelist.push({item:"pretax", tips: "项目未税金额", price: this.computedSkuPricePretax, discount: "--"})
        pricelist.push({item:"tax", tips: "税金", price: this.computedSkuPriceTax, discount: this.skuDiscount.tax})
        pricelist.push({item:"total", tips: "总概算：人民币圆整", price: this.computedSkuPriceTotal, discount: this.skuDiscount.total})

        return pricelist
      },
      showSkuIcon(node) {
        this.operation = "showSkuIcon"

        node.data.d_set = "show"

        let _src = ''
        let icon_id = this.Icons.findIndex(d => d.id === node.data.icon_id)
        if (icon_id >= 0) {
          _src = this.Icons[icon_id].src
          this.Icons[icon_id].d_set = "show"
        }

        let _index = this.LoadIcons.findIndex(d => d.src == _src);
        if (_index == -1 && _src) {
          this.LoadIcons.push({src: _src, load: false});
        }
      },
      async doClickSkuProduct(sku, stashModel = false, parentId = 0, productName = '', setNum = 1) {

        if (stashModel === false) {
          this.stashImageCoords.left = 0
          this.stashImageCoords.top = 0
        }

        if (this.currNode_id > 0 && this.currNode_pid > 0) {

          this.querySkuImageLoading = true;

        	let _i = this.querySkuParams.fid
        	let _j = this.querySkuParams.sid
        	let _k = this.querySkuParams.tid

          let that = this
      	  let tab_id = _k
      	  let tab_lable = ''
          let i_lable = ''
          let j_lable = ''

          try{

        	 this.categoryList.forEach(function(val,index,array){
                if (val.value == _i && val.children) {
                    val.children.forEach(function(_val,_index,_array){
		                if (_val.value == _j && _val.children) {
		                    _val.children.forEach(function(__val,__index,__array){
				                if (__val.value == _k) {
                            i_lable = val.label
                            j_lable = _val.label
				                    tab_lable = __val.label
				                }
				              })
		                }
		              })
                }
            })

            if (i_lable == '' || j_lable == '') {
              const res = await openApi.doQueryProductCategory({token: this.TOKEN}, { pid: sku.pid})
              if (res.data.code == 200 && res.data.data.types) {
                let types = res.data.data.types[0]
                i_lable = types.typeonename
                j_lable = types.typetwoname
                tab_lable = types.typethrname
              }
            }

            let fid = this.FID;
            let _label = sku.label;
            let _code = sku.id || Number(new Date());
            let icon_id = '';

            if (this.Icons[0].op_index) {
              this.Icons[0].op_index += 1;
              icon_id = _code + '-' + this.Icons[0].op_index;
            } else {
              let seq_id = parseInt(new Date().getTime()/1000);
              icon_id = _code + '-' + seq_id;
              this.Icons[0].op_index = seq_id;
              //icon_id = _code + '-' + this.Icons.length;
            }

            let max_id = 0
            let temp_ids = []
            let sku_img = ''
            let sku_img_i = ''
            let sku_img_m = ''
            let exist = false
            let d_set = this.proSetting.createModel == 'drawing' ? "show" : "hide"

            this.skuProductList.some(function(val,index,array){

                if (val.id == sku.id) {

                    sku_img = val.image
                    sku_img_i = val.sku_img_i
                    sku_img_m = val.sku_img_m

                    let discount = parseInt(array[index].discount);

                    array[index].num += setNum
                    if (array[index].set_num) {
                      array[index].set_num += setNum
                    } else {
                      array[index].set_num = array[index].num
                    }

                    let f_index = -1
                    if (typeof(array[index].fids) != "undefined" && (array[index].fids instanceof Array)) {
                      f_index = array[index].fids.findIndex(d => d.fid === fid);
                    } else {
                      f_index = -1
                      array[index].fids = []
                    }

                    if (f_index >= 0) {
                      array[index].fids[f_index].num += setNum

                      array[index].fids[f_index].rids ? array[index].fids[f_index].rids.push(that.currNode_id) : array[index].fids[f_index].rids = [that.currNode_id]
                    } else {
                      array[index].fids.push({fid: fid, num: setNum, rids: [that.currNode_id]})
                    }

                    if(discount >= 0 && discount <= 10000) {
                      array[index].total_money = parseFloat(array[index].price * array[index].set_num * discount / 100).toFixed(2)
                    }else{
                      array[index].total_money = parseFloat(array[index].price * array[index].set_num).toFixed(2)
                    }

                    if (parentId > 0 && typeof(val.parent_ids) != "undefined" && (val.parent_ids instanceof Array)) {
                      val.parent_ids.push(parentId)
                    }
                    if (parentId > 0 && typeof(val.parent_nums) != "undefined" && (val.parent_nums instanceof Array)) {
                      val.parent_nums.push({"pid":parentId,"num":setNum})
                    }

                    exist = true
                    return true
                }
            })

            let sync_img = ''
            let sync_mini_img = ''
            let sync_medium_img = ''

            if (exist === false) {
              if (sku.item != '默认规格') {
                const res = await openApi.doQuerySkuImage({token: this.TOKEN}, {coloritem: sku.item, pid: sku.pid, cache: this.querySkuParams.cache})
                if (res.data.code == 200) {
                  sync_img = res.data.data.src
                  sync_mini_img = res.data.data.src_i
                  sync_medium_img = res.data.data.src_m
                }
              }

              sku_img = sync_img || sku.base.image_thum || sku.base.image
              sku_img_i = sync_mini_img || sku.base.image_thum_i
              sku_img_m = sync_medium_img || sku.base.image_thum_m


              let mini_discount = 0
              if (sku.price > 0 && sku.low_price > 0) {
                mini_discount = parseInt(sku.low_price / sku.price * 100);
              }

              let discount = 100
              let total_money = sku.price || 0

              if (isNaN(total_money)) {
                total_money = 0
              }

              let skuNode = {
                id: sku.id,
                pid: sku.pid,
                code: sku.code,
                label: sku.label,
                item: sku.item,
                price: sku.price,
                sale_price: sku.sale_price,
                trade_price: sku.trade_price,
                channel_a: sku.channel_a,
                channel_b: sku.channel_b,
                channel_c: sku.channel_c,
                low_price: sku.low_price,
                cost_price: sku.cost_price,
                mini_discount: mini_discount,
                model_name: sku.base.model_name,
                type_name: sku.base.type_name,
                brand_name: sku.base.brand_name,
                image: sku_img,
                sku_img_i: sku_img_i,
                sku_img_m: sku_img_m,
                unit: sku.base.unit,
                desc: sku.base.desc,
                num: 1,
                set_num: 1,
                total_money: total_money,
                discount: discount,
                i: _i,
                j: _j,
                k: _k,
                i_lable: i_lable,
                j_lable: j_lable,
                k_lable: tab_lable,
                parent_ids: [],
                parent_nums: [],
                fids: [{"fid":fid, "num":1, 'rids':[this.currNode_id]}]
              };

              if (setNum > 1) {
                skuNode.num = setNum;
                skuNode.set_num = setNum;
                let rids = []
                //let pids = []
                for (let ri = 0; ri < setNum; ri++) {
                  rids.push(this.currNode_id)
                  //if (parentId) pids.push(parentId)
                }

                //skuNode.parent_ids = pids
                skuNode.fids = [{"fid":fid, "num":setNum, 'rids':rids}];
              }

              if (parentId) {
                skuNode.parent_ids.push(parentId)
                skuNode.parent_nums.push({"pid":parentId,"num":setNum})
              }

              this.skuProductList.push(skuNode);
            }

            for (let x = 0; x < setNum; x++) {
              if (parentId == 0) {
                let iconNode = {
                    id: this.currNode_id,
                    pid: this.currNode_pid,
                    type: 2,
                    code: tab_id,
                    label: tab_lable,
                    sku_label: sku.label,
                    proid: sku.id,
                    d_set: d_set,
                };

                this.Icons.push({
                    id: icon_id,
                    src: sku_img,
                    src_i: sku_img_i,
                    src_m: sku_img_m,
                    model: this.proSetting.thumbModel,
                    d_set: d_set,
                    node: iconNode,
                    load: 1,
                    data: {}
                });

                this.pushProjectNode(iconNode, icon_id);
              }

              this.upTagProductList(tab_id, tab_lable, 'add', 2);

              let skuCateLable = i_lable + '/' + j_lable;
              this.checkedListPush(skuCateLable)
            }

            app.$message({
              message: '产品添加成功',
              type: 'success'
            });
          }catch(e) {
            console.log(e)
          }

          this.querySkuImageLoading = false;
        } else {
           this.$message({
                message: '请先勾选左侧菜单栏的楼层和房间',
                type: 'warning'
           });
        }
      },
      upSkuPrice(){
        let usedPrice = this.proSetting.usedPrice
        this.skuProductList.some(function(val,index,array){
            let calc_price = 0
            if (usedPrice == 'trade_price' && typeof(val.trade_price) != "undefined") {
              calc_price = val.trade_price
            } else if (usedPrice == 'channel_a' && typeof(val.channel_a) != "undefined") {
              calc_price = val.channel_a
            } else if (usedPrice == 'channel_b' && typeof(val.channel_b) != "undefined") {
              calc_price = val.channel_b
            } else if (usedPrice == 'channel_c' && typeof(val.channel_c) != "undefined") {
              calc_price = val.channel_c
            } else if (usedPrice == 'sale_price' && typeof(val.sale_price) != "undefined") {
              if (val.low_price && val.low_price > 0 && val.low_price > val.sale_price) {
                calc_price = val.low_price
              } else {
                calc_price = val.sale_price
              }
            } else {
              calc_price = 0
            }

            let discount = parseInt(val.discount);

            if(discount >= 0 && discount <= 10000) {
              array[index].total_money = parseFloat(calc_price * val.set_num * discount / 100).toFixed(2)
            }else{
              array[index].total_money = parseFloat(calc_price * val.set_num).toFixed(2)
            }

            array[index].price = calc_price
        })

        if (this.costWorking.type == 'category') {
          this.doUpdateCategoryCost()
        } else {
          this.computedCostWorking()
        }
      },
      pushCateProductList(skuNode, hasCheckedList = false) {

        let _exist = false;
        let _existIndex = -1;
        let skuCateLable = skuNode.i_lable + '/' + skuNode.j_lable;

        this.skuCateProductList.some(function(val,index,array){
          if (skuCateLable == val.label) {
            _existIndex = index;
          }
          if (_exist === false) {
            val.sku.some(function(_val,_index,_array){
              if (_exist === false && _val.id == skuNode.id) {
                  //_array[_index].num += 1
                  let _discount = parseInt(_array[_index].discount);
                  let __num = _array[_index].set_num || _array[_index].num
                  if(_discount >= 0 && _discount <= 10000) {
                    _array[_index].total_money = parseFloat(_array[_index].price * __num * _discount / 100).toFixed(2)
                  }else{
                    _array[_index].total_money = _array[_index].price * __num
                  }
                  _exist = true
                  return true

              }
            })
          }

          if (_exist === true) {
            return true
          }
        })

        if (!_exist) {
            if (_existIndex != -1) {
              this.skuCateProductList[_existIndex].sku.push(skuNode);
            }else{
              this.skuCateChoose.push(skuCateLable)

              if (hasCheckedList === false) {
                this.checkedListPush(skuCateLable)
              }

              this.skuCateProductList.push({
                label: skuCateLable,
                sku: [skuNode]
              });

              if (this.categoryCost.item.findIndex(d => d.label == skuCateLable) == -1) {
                this.categoryCost.item.push({
                  label: skuCateLable,
                  value: 0.00
                })
              }
            }
        }
      },
      doClickTagProduct(data) {

        if ( this.uploadBGImage === false) {
           this.$message({
                message: '请先上传当前编辑楼层的工程图',
                type: 'warning'
           });
          return;
        }

        if (!this.currNode_id || this.currNode_id.toString().length < 5) {
           this.$message({
                message: '请正确勾选当前楼层和房间节点',
                type: 'warning'
           });
           return
        }

        let d_set = this.proSetting.createModel == 'drawing' ? "show" : "hide"

        if(data.level == 3) {
            if (this.currNode_id > 0 && this.currNode_pid > 0) {

                let _label = data.label;
                let _code = data.value || '';
                let icon_id = '';

                if (this.Icons[0].op_index) {
                  this.Icons[0].op_index += 1;
                  icon_id = _code + '-' + this.Icons[0].op_index;
                } else {
                  let seq_id = parseInt(new Date().getTime()/1000);
                  this.Icons[0].op_index = seq_id;
                  icon_id = _code + '-' + seq_id;
                  //icon_id = _code + '-' + this.Icons.length;
                }

                let iconNode = {
                    id: this.currNode_id,
                    pid: this.currNode_pid,
                    type: 1,
                    code: _code,
                    label: _label,
                    proid: data.value,
                    d_set: d_set,
                };

                this.Icons.push({
                    id: icon_id,
                    src: data.image_thum,
                    src_i: data.image_thum_i,
                    src_m: data.image_thum_m,
                    model: this.proSetting.thumbModel,
                    node: iconNode,
                    d_set: d_set,
                    load: 1,
                    data: {}
                });

                this.pushProjectNode(iconNode, icon_id);
                this.upTagProductList(_code, _label, 'add', 1);

                //this.$nextTick(() => {
                //    CanvasDemo.addIcon(_id, {});
                //});

            } else {
                 this.$message({
                      message: '请先勾选左侧菜单栏的楼层和房间',
                      type: 'warning'
                 });
            }

        }
      },

      openLoading() {
        this.loading = this.$loading({
          lock: true,
          text: 'Loading',
          spinner: 'el-icon-loading',
          background: 'hsla(0,0%,100%,.9)'
        });
      },
      handleDiscountChange(value, id) {
        let sku = this.skuProductList.find(i => i.id == id)
        let index = this.skuProductList.findIndex(i => i.id == id)
        if(value >= 0 && value <= 100) {
          let _num = sku.set_num || sku.num
          this.skuProductList[index].discount = value
          this.skuProductList[index].total_money = parseFloat(sku.price * _num * value / 100).toFixed(2)
        }
      },
      handleNumChange(value, id) {
        let sku = this.skuProductList.find(i => i.id == id)
        let index = this.skuProductList.findIndex(i => i.id == id)
        if(value >= 0 && value <= 9999) {
          this.skuProductList[index].set_num = value
          this.skuProductList[index].total_money = parseFloat(sku.price * value * sku.discount / 100).toFixed(2)
        }
      },
      handlePriceChange(value, id) {
        let sku = this.skuProductList.find(i => i.id == id)
        let index = this.skuProductList.findIndex(i => i.id == id)
        if(value >= 0) {
          this.skuProductList[index].price = value
          this.skuProductList[index].total_money = parseFloat(value * sku.set_num * sku.discount / 100).toFixed(2)
        }
      },
      handleProDiscountChange(value, item) {
        return true
      },
      upTreeNode(id, data) {
        this.projectData.some((val,index,array)=>{
          val.children.some((v,i,a)=>{
            v.children.some((_v,_i,_a)=>{
              if (_v.icon_id == id && data.d_set) {
                _v.d_set = data.d_set
                return true;
              }
            })
          })
        })
      },
      getTreeNode(id, level = 1) {
            let node = {id: 0, label: ''};
            if(this.projectData.length == 0) {
                return node;
            }
            this.projectData.forEach(function(val,index,array){
                if (level == 1) {
                    if (val.id == id) {
                       node.id = val.id;
                       node.label = val.label;
                       return;
                    }
                } else if (level == 2) {
                    val.children.forEach(function(_val, _index, _array){
                        if (_val.id == id) {
                           node.id = _val.id;
                           node.label = _val.label;
                           return;
                        }
                    })
                } else if (level == -1) {
                    //找children第一个node的id
                    if (val.id == id) {
                        node.id = val.children[0].id || 0;
                        node.label = val.children[0].label || '';
                        return;
                    }
                }
            })

            return node;
      },
      pushProjectNode(params, icon_id) {
        try {
          let _i = this.projectData.findIndex(d => d.id == params.pid);
          let _j = this.projectData[_i].children.findIndex(d => d.id == params.id);
          let projectNode = this.projectData[_i].children[_j];
          let childrenLen = projectNode.children.length;
          let _label = params.sku_label || params.label;
          let d_set = params.d_set || "show";

          if (typeof(this.projectData[_i].children) == 'undefined') {
            //console.log(_i, _j, params, icon_id)
          }

          let seq_id = icon_id.split("-")[1]

          if (typeof(seq_id) == 'undefined') {
            seq_id = parseInt(new Date().getTime()/1000);
          }

          projectNode.children.push({
              id: projectNode.id + '-' + seq_id,
              icon_id: icon_id,
              value: params.code,
              del: true,
              d_set: d_set,
              disabled: childrenLen == 0 ? false : true,
              type: params.type,
              label: _label
          });
        } catch(e) {
          console.log(e)
          //console.log(params, icon_id)
        }
      },
      async loadProjectRoomData() {
        this.openLoading();
        const res = await dashboardApi.doQueryRoom({pid:this.PID, fid:0, token: this.TOKEN})
		    try {
          if (res.data.code == 200) {
	          //产品列表数据
	          this.projectData = res.data.data

            if (this.Icons && this.Icons.length > 0) {
              this.pushProjectIcon();
            }
          } else {
          	console.log(res.data.msg)
          }
          this.loading.close();
        } catch (e) {
          this.loading.close();
          console.log(e)
        }
      },
      async doChangeSkuBrandList() {
          this.doQuerySkuProduct()
      },
      async doChangeSkuModelList() {
          this.querySkuParams.mid = this.selectedModel[1] || ''
          let heckedNodes = this.$refs.selectedModelCascader.getCheckedNodes()

          if(typeof(heckedNodes[0]) != 'undefined') {
            this.querySkuParams.lable = heckedNodes[0].label || ''
          }

          if (this.querySkuParams.mid != '') {
            this.doQuerySkuProduct()
          }
      },
      async doChangeSkuCategoryList() {
          if (this.$refs.refHandle) {
            this.$refs.refHandle.dropDownVisible = false; //监听值发生变化就关闭它
          }
          this.querySkuParams.fid = this.selectedCategory[0] || ''
          this.querySkuParams.sid = this.selectedCategory[1] || ''
          this.querySkuParams.tid = this.selectedCategory[2] || ''

          let i = this.selectedCategory.length
          let id = i > 0 ? this.selectedCategory[i - 1] : 0

          if (id) {
            this.dodQueryBrandInfo(id)
            this.dodQuerySpecInfo(id)
          }

          if (this.querySkuParams.tid != '') {
            this.doQuerySkuProduct()
          }
      },
      async doClearCategory(refresh = false) {
        //const res = await dashboardApi.doClearCategory({token: this.TOKEN})
        const res = await openApi.doQueryCategory({token: this.TOKEN, refresh: refresh})
        try {
          if (res.data.code == 200) {
            this.categoryList = res.data.data
            this.tagProduct = res.data.data
          }
        } catch (e) {
          console.log(e)
        }
      },
      async getQRCode(type) {
        const res = await dashboardApi.doQRCode({token: this.TOKEN, pid: this.PID, type: type})

        try {
          if (res.data.code == 200) {
            if (type == 'cateSkuListQR') {
              this.cateSkuListQR = res.data.data.url + "?_r=20200502"
            } else if (type == 'roomSkuListQR') {
              this.roomSkuListQR = res.data.data.url + "?_r=20200502"
            } else if (type == 'albumsQR') {
              this.albumsQR = res.data.data.url + "?_r=20200502"
            }
          }
        } catch (e) {
          console.log(e)
        }
      },
      async dodQueryBrandInfo(id) {
        let post = JSON.parse(JSON.stringify(this.querySkuParams))
        post.action = 'queryBrand'
        const res = await openApi.dodQueryBrandInfo({token: this.TOKEN}, post)

        try {
          if (res.data.code == 200) {
            this.brandSelected.splice(0, this.brandSelected.length)
            res.data.data.map((val,index,array) => {
              Vue.set(this.brandSelected, index, val)
            })
          }
        } catch (e) {
          console.log(e)
        }
      },
      async dodQuerySpecInfo(id) {
        let post = JSON.parse(JSON.stringify(this.querySkuParams))
        post.action = 'querySpec'
        const res = await openApi.dodQueryBrandInfo({token: this.TOKEN}, post)

        try {
          if (res.data.code == 200) {
            this.modelSelected.splice(0, this.modelSelected.length)
            res.data.modelData.map((val,index,array) => {
              Vue.set(this.modelSelected, index, val)
            })
          }
        } catch (e) {
          console.log(e)
        }
      },
      async handleUserCommand(val) {
        if (val == 'setting') {
          this.settingFormVisible = true
        } else if (val == 'logout') {
          await this.doLogout()
        }
      },
      async doSaveSetting() {
        let upCostWorking = this.updateCostSetting ? 1 : 0
        const res = await dashboardApi.doSaveSetting({token: this.TOKEN,pid: this.PID, tag: upCostWorking}, this.costSetting)
        try {
          if (res.data.code == 200) {
              if (this.costWorking) {
                this.costWorking.normalCost = this.costSetting
                this.costWorking.categoryCost = []
                this.computedCostWorking()
                this.doUpdateCategoryCost()
              }
              this.$message({
              type: 'success',
              message: '保存成功'
            });
            this.settingFormVisible = false
          } else {
            console.log(res.data.msg)
          }
        } catch (e) {
          console.log(e)
        }
      },
      async doSaveCostWorking() {
        const res = await dashboardApi.doSaveCostWorking({token: this.TOKEN}, this.costWorking)
        try {
          if (res.data.code == 200) {
                this.$message({
                type: 'success',
                message: '保存成功'
            });
            this.settingFormVisible = false
          } else {
            console.log(res.data.msg)
          }
        } catch (e) {
          console.log(e)
        }
        this.costCategoryVisible = false
      },
      async doLogout(closeWindow = false) {

        if (closeWindow) {
          await dashboardApi.doLogout({pid:this.PID, fid:this.FID, token: this.TOKEN})
          return
        }

        this.$confirm('确定要退出系统吗?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(async () => {
            const res = await dashboardApi.doLogout({pid:this.PID, fid:this.FID, token: this.TOKEN})
            if (res.data.code == 200) {
                this.$message({
                  message: '退出成功',
                  type: 'success',
                  onClose: () => {
                    closeWindow()
                  }
                });
            }
        }).catch(() => {

        });
      },
      async doLive() {
          const res = await dashboardApi.doLive({pid:this.PID, fid:this.FID, token: this.TOKEN})
          if (res.data.code != 200) {
              this.$message({
                message: '网络异常，请检查本地网络链接，或刷新本页面',
                type: 'warning',
              });
          }
      },
     configureModelDialog() {
        this.configureModelVisible = true
        this.configureModelBtnVisible = true
     },
     async configureModelSubmit(model) {

        var init_fid = 17

        this.proSetting.createModel = model
        this.configureModelBtnVisible = false
        if (model == 'drawing') {
          this.mainTabsActive = 'canvasTab'
        } else if (model == 'writing') {
          this.mainTabsActive = 'skuListTab'
        }

        const res = await dashboardApi.updateSetting({pid:this.PID, fid:init_fid, key:"createModel", val:model, token: this.TOKEN}, {})

        if(model == "writing"){
          const res2 = await dashboardApi.doSaveRoom({pid: this.PID, fid:init_fid, token: this.TOKEN}, [101])
        }

        setTimeout(()=>{
          if(model == "writing"){
            this.redirect(init_fid);
          }

          this.configureModelVisible = false
          this.$notify({
            title: '提示',
            type: 'warning',
            message: '请先添加项目楼层房间数据',
            duration: 2000
          });

          if (this.uploadBGImage === false && model == "drawing") {
            setTimeout(()=>{
              this.$notify({
                  title: '提示',
                  type: 'warning',
                  message: '请先上传当前编辑楼层的工程图',
                  duration: 2000
                });
             }, 1000)
             this.loading.close();
          }
          this.autoCommitSwitch = true
        }, 200)

     },
     async getHomeData(id) {
        this.openLoading();

        const res = await dashboardApi.getHomeData({pid:this.PID, fid:this.FID, token: this.TOKEN})

        document.getElementById('canvasdemo').parentNode.parentNode.style.height = (this.FullScreenHeight - 40) + 'px';

        try {
          if (res.data.code == 506) {
            this.loading.close();
            this.autoCommitSwitch = false
            this.$confirm(res.data.msg, '系统提示', {
              type: 'warning',
              showClose: false,
              showConfirmButton: false,
              showCancelButton: false,
              closeOnClickModal: false,
              closeOnPressEscape: false,
              closeOnHashChange: false,
            });
            return
          }else if (res.data.code == 200) {

            //版权信息
            this.copyRight = res.data.data.copyRight
            //公司信息
            this.companyInfo = res.data.data.companyInfo
            //制图Icons数据集合
            this.Icons = res.data.data.icons
          	//项目基本数据
            this.baseProjectData = res.data.data.baseProductProData
            //产品清单折扣数据
            this.skuDiscount = res.data.data.skuDiscount
            //产品清单分类筛选数据
            this.checkedList = res.data.data.checkedList
            //产品清单分类工程费用数据
            this.categoryCost = res.data.data.categoryCost
            //项目楼层房间数据
            this.projectData = res.data.data.projectData
            //产品列表数据
            this.skuProductList = res.data.data.skuProductList
            //方案清单数据排序
            this.skuCateSort = res.data.data.skuCateSort
            //右侧边栏产品分类数据
            //this.categoryList = res.data.data.category
            //右侧边栏功能三级联动选择器数据
            //this.tagProduct = res.data.data.tagProduct
            //已选的功能汇总清单
            this.tagProductList = res.data.data.tagProductList
            //项目楼层数据
            this.floorSelect = res.data.data.floorConfig
            //项目房间数据
            this.roomSelect = res.data.data.roomConfig

            //品牌列表数据
            //this.brandSelected = res.data.data.brandData

            //项目设置数据
	          //this.proSetting = Object.assign({}, this.proSetting)
            this.proSetting = res.data.data.proSetting
            //this.proSetting.createModel = "drawing"

            this.costSetting = res.data.data.costSetting

            this.costWorking = res.data.data.costWorking


            let getModel = getUrlParams('model')
            let apiModel = res.data.data.proSetting.thumbModel

            if (getModel == null) {
              this.proSetting.thumbModel = apiModel
              this.redirect(this.FID)
            }

            if (this.proSetting.createModel == "") {
              this.configureModelDialog()
            } else {
              this.autoCommitSwitch = true

              if(this.projectData == null || this.projectData.length == 0) {
                  this.$notify({
                    title: '提示',
                    type: 'warning',
                    message: '请先添加项目楼层房间数据',
                    duration: 2000
                  });
              }

              if (this.proSetting.createModel == 'drawing') {
                this.mainTabsActive = 'canvasTab'
              } else if (this.proSetting.createModel == 'writing') {
                this.mainTabsActive = 'skuListTab'
              }
            }

            if (this.checkedList.length > 0) {
              this.hasCheckedList = true
            }

            //打印设置数据
            if (res.data.data.printForm.length > 0) {
              this.printForm = res.data.data.printForm
            }

            if (typeof(this.skuDiscount.items) == 'undefined') {
              this.skuDiscount.items = {const_tips: '辅材及施工', design_tips: '设计与安装调试费'}
            }

            if (this.skuProductList.length == 0) {
              this.skuProductListEmpty = true
            }

            let that = this

            /*this.categoryList.forEach(function(val,index,icon){
                that.fSkuCategroy.push({
                  value: val.value,
                  label: val.label
                });
            })*/

            /*if (this.skuCateProductList.length == 0) {
              this.skuProductList.forEach((val,index,array)=>{
                this.pushCateProductList(val, hasCheckedList);
              })
            }*/

            /*this.skuProductList.forEach((val,index,array)=>{
              this.pushCateProductList(val, hasCheckedList);
            })*/

            this.currNode_pid = this.FID;
            let firstNode = this.getTreeNode(this.FID, -1);
            if (firstNode.id) {
                this.currNode_id = parseInt(firstNode.id);
                this.$refs.productTree.setCheckedKeys([firstNode.id]);
            }

            this.LoadIcons = [];
            if (this.showIcons && this.showIcons.length > 0) {
              this.showIcons.forEach(function(val,index,icon){
                let _index = that.LoadIcons.findIndex(d => d.src === val.src);
                if (_index == -1) {
                  that.LoadIcons.push({src: val.src, load: false});
                }
              });
              this.pushProjectIcon();
            }

            if (this.uploadBGImage === false && this.proSetting.createModel == "drawing") {
              setTimeout(()=>{
                this.$notify({
                    title: '提示',
                    type: 'warning',
                    message: '请先上传当前编辑楼层的工程图',
                    duration: 2000
                  });
               }, 1000)
            }

            if (this.costWorking.type == 'category') {
              this.doUpdateCategoryCost()
            }

          this.getQRCode('cateSkuListQR');
          this.getQRCode('roomSkuListQR');

           this.loading.close();
          } else {
            this.loading.close();
            this.$message.error(res.data.msg);
          }

        } catch (e) {
          console.log(e)
          this.loading.close();
        }
      },
      pushProjectIcon() {

        let that = this

        this.Icons.forEach(function(val,index,icon){
           let isBGImage = false
           if (val.src.indexOf("-BG") > 0) {
              isBGImage = true
              that.uploadBGImage = true
            }
            if (val.src.indexOf("http") == -1) {
              val.src = process.env.BASE_URL + val.src
            }
            if (val.node && val.node.id && val.node.pid && isBGImage === false) {
                that.pushProjectNode({
                    pid: val.node.pid,
                    id: val.node.id,
                    code: val.node.code,
                    label: val.node.label,
                    sku_label: val.node.sku_label || '',
                    type: val.node.type,
                    d_set: val.d_set || 'show',
                }, val.id);
            }
        });
      },
      loadImage(icon) {
        if (this.LoadIcons.length > 0) {
          let _index = this.LoadIcons.findIndex(d => d.src === icon.src);
          if (_index >= 0) {
            this.LoadIcons[_index]['load'] = true;
            icon.load = 1;
          }
        }

        if (this.initFinish === false) {

          let loadFinish = true;
          this.LoadIcons.some(function(val,index,icon){
            if (val.load === false) {
              loadFinish = false;
              return true;
            }
          });

          if (loadFinish === true) {
            CanvasDemo.init();
            this.loading.close();
            this.initFinish = true;
            //console.log("loadFinish icons...", this.Icons)
          }
        } else {
          //let iconSize = this.iconSize[this.proSetting.thumbModel]
          //this.$refs[icon.id][0].width = iconSize.width || 40
          //this.$refs[icon.id][0].height = iconSize.height || 40
          if (icon.load && icon.load == 1) {
            if (this.operation == "showSkuIcon") {
              CanvasDemo.addIcon(icon.id, icon.data);
              this.operation = "";
            }else if (this.stashImageCoords.left == 0 && this.stashImageCoords.top == 0) {
              CanvasDemo.addIcon(icon.id, {});
            } else {
              CanvasDemo.addIcon(icon.id, this.stashImageCoords);
            }

            let iconIndex = this.Icons.findIndex(d => d.id === icon.id);
            if (iconIndex >= 0) {
                this.Icons[iconIndex]['load'] = 0;
            }
          }
          //CanvasDemo.renderCanvas()
        }
      },

     async getSkuProductList(pid, fid) {
        const res = await dashboardApi.getSkuProductList({pid:pid, fid:fid, token: this.TOKEN})
        try {
          if (res.data.code == 200) {
	          //产品列表数据
            this.skuProductList = res.data.data
          } else {
          	console.log(res.data.msg)
          }
        } catch (e) {
          console.log(e)
        }
      },

      async floorChange(id) {
        const res = await dashboardApi.doQueryRoom({pid:this.PID, fid: this.floorSelected, token: this.TOKEN})
        try {
          if (res.data.code == 200) {
	          //产品列表数据
	          this.roomsSelected = res.data.data
          } else {
          	console.log(res.data.msg)
          }
        } catch (e) {
          console.log(e)
        }
      },
      async doSubmitFloorAdd() {
        let data = {label: this.addFloorName}
        const res = await dashboardApi.doSaveExtendsRoom({pid: this.PID, token: this.TOKEN}, data)

        try {
          if (res.data.code == 200) {
            //this.loadProjectRoomData();
            this.roomSelect.unshift(res.data.data)
            this.roomsSelected.push(res.data.data.id)
          } else {
          	console.log(res.data.msg)
          }
        } catch (e) {
          console.log(e)
        }
      },
      async doSubmitFloorEdit(id) {
        let fid = this.floorSelected;
        let data = JSON.stringify(this.roomsSelected)
        const res = await dashboardApi.doSaveRoom({pid: this.PID, fid: fid, token: this.TOKEN}, data)

        try {
          if (res.data.code == 200) {
            this.loadProjectRoomData();
          } else {
          	console.log(res.data.msg)
          }
        } catch (e) {
          console.log(e)
        }
      },
      changeLabel(prop) {
        let id = prop.id
        let label = prop.label
        this.$refs[`${prop.id}`].blur()
        if(typeof(prop.type) == 'undefined') {
          this.$prompt('请输入别名', '提示', {
            inputValue: label,
            confirmButtonText: '确定',
            cancelButtonText: '取消',
          }).then(({ value }) => {
            this.postlabelEdit(id,value)
          }).catch(() => {
            this.$message({
              type: 'info',
              message: '取消输入'
            });
          });
        }
      },
      async postlabelEdit(id,value) {
        const res = await dashboardApi.doSaveAliasRoom({pid:this.PID, token: this.TOKEN}, {id: id, label: value})
        try {
          if (res.data.code == 200) {
                this.$message({
                type: 'success',
                message: '修改成功'
            });
            this.loadProjectRoomData();
          } else {
          	console.log(res.data.msg)
          }
        } catch (e) {
          console.log(e)
        }
      },
      async doSaveImageCoords(data, auto = false) {
      	let _data = CanvasDemo.saveData();
        //_data.icons.splice(_data.icons.findIndex(i => i.id == "Copyright-Icon"),1)

        if (this.skuProductListEmpty === false && (_data.skuList == null || _data.skuList.length == 0)) {
          if (auto) return;
           this.$alert('方案清单数据异常, 终止提交！', '系统异常', {
            confirmButtonText: '确定',
            callback: action => {
              this.$message({
                type: 'info',
                message: "已取消提交"
              });
            }
          });
          return;
        }

        const res = await dashboardApi.doSaveImageCoords({pid:this.PID, fid: this.FID, token: this.TOKEN}, _data)
        try {
          if (res.data.code == 200) {
             this.$message({
                  message: auto ? '数据自动保存成功' : '数据保存成功',
                  type: 'success'
             });
          } else {
             this.$message({
                  message: res.data.msg,
                  type: 'error'
             });
          }
        } catch (e) {
          console.log(e)
        }
      },
      redirect(fid) {
        let thumbModel = this.proSetting.thumbModel || 'small'
        if (fid) {
          window.location.href = "/dist/index.html?pid=" + this.PID + "&fid=" + fid + "&model=" + thumbModel + "&token=" + this.TOKEN;
        } else {
          window.location.href = "/dist/index.html?pid=" + this.PID + "&model=" + thumbModel + "&token=" + this.TOKEN;
        }
      },
      redirectPrint(cache_key) {
        window.open("/Gateway/printCanvas/"+this.PID+"/"+ this.FID +"?token=" + this.TOKEN + "&cache_key=" + cache_key);
      },
      async initPDF() {
        // 生成图片
        this.imageBuffer = []
        let pages = document.getElementsByClassName("skuListPage")
        for (let i = 0; i < pages.length; i++) {
          let currentPage = pages[i]
          let canvas = await html2canvas(currentPage, {
            scale: 2
          })
          let pageData = canvas.toDataURL('image/jpeg', 1);
          this.imageBuffer.push(pageData)
        }

        // 生成pdf预览
        this.doc = new jsPDF('', 'pt', 'a4')
        for (let i = 0; i < this.imageBuffer.length; i++) {
          i && this.doc.addPage()
          this.doc.addImage(this.imageBuffer[i], 'JPEG', 0, 0, 594.3, 840.51);
        }
        this.pdf = this.doc.output('bloburi');
      },
      async initPDFLandscape() {
        // 生成图片
        this.imageBufferL = []
        let pages = document.getElementsByClassName("skuListPageLandscape")
        for (let i = 0; i < pages.length; i++) {
          let currentPage = pages[i]
          let canvas = await html2canvas(currentPage, {
            scale: 1
          })
          let pageData = canvas.toDataURL('image/jpeg', 1);
          this.imageBufferL.push(pageData)
        }
        this.docL = new jsPDF('l', 'pt', 'a4')
        for (let i = 0; i < this.imageBufferL.length; i++) {
          i && this.docL.addPage()
          this.docL.addImage(this.imageBufferL[i], 'JPEG', 0, 0, 840.51, 594.3);
        }
        this.pdfL = this.docL.output('bloburi');
      },
      async initPDFCanvas() {
        // 生成图片
        this.imageBufferC = []

        this.$nextTick(async () => {
          let pages = document.getElementsByClassName("printCanvasPage")
          for (let i = 0; i < pages.length; i++) {
            let currentPage = pages[i]
            let canvas = await html2canvas(currentPage, {
              scale: 2
            })
            let pageData = canvas.toDataURL('image/jpeg', 0.92);
            this.imageBufferC.push(pageData)
          }

          // 生成pdf预览
          //this.docC = new jsPDF('l', 'pt', 'a4')
          this.docC = new jsPDF('l', 'pt', 'a3')
          for (let i = 0; i < this.imageBufferC.length; i++) {
            i && this.docC.addPage()
            //this.docC.addImage(this.imageBufferC[i], 'JPEG', 0, 0, 840.51, 594.3);
            this.docC.addImage(this.imageBufferC[i], 'JPEG', 0, 0, 1188.1,840);
          }
          this.pdfC = this.docC.output('bloburi');
        });
      },
      viewpdf(action, type) {
        if(!this.pdf && !this.pdfC) return
        let tempwindow
        if (action == "preview") {
          tempwindow = window.open('_blank') // 先打开新页面防止被拦截

          if (type == 'canvasPrint') {
            tempwindow.location = this.pdfC; // 预览pdf
          } else {
            tempwindow.location = this.pdf; // 预览pdf
          }

        } else if (action == "save") {
          let doc_name;
          if (type == 'canvasPrint') {
            doc_name = '点位图';
            this.docC.save(this.baseProjectData.businessname + "-" + doc_name + ".pdf"); // 下载pdf
          } else {
            doc_name = '产品清单';
            this.doc.save(this.baseProjectData.businessname + "-" + doc_name + ".pdf"); // 下载pdf
          }
        }
      },
      tagProductListFilter(type){
        return this.tagProductList.filter(function(item){
          return item.type == type
        })
      },
      beforeunloadHandler(){
        this._beforeUnload_time = new Date().getTime();
      },
      unloadHandler(e){
        this._gap_time = new Date().getTime()-this._beforeUnload_time;
        if(this._gap_time<=5){
          this.doLogout(true)
        }
      },
      getCategoryTotal(category) {
        let basePrice = 0
        this.skuProductList.forEach((val,index,array) => {
          let _lable = val.i_lable + '/' + val.j_lable;
          let i = category.findIndex(d => d == _lable)
          if (i > -1) {
            basePrice += parseFloat(val.total_money);
          }
        })
        return basePrice
      },
      doUpdateCategoryCost(){
        this.computedSkuCateLable.forEach((val,index,array)=>{
          let cate_id = this.costWorking.categoryCost.findIndex(d => d.category_real_text == val)
          if (cate_id == -1) {
            let defaultCost = JSON.parse(JSON.stringify(this.computedOepnCostList))
            defaultCost[0].item = '设备小计'
            defaultCost[defaultCost.length-1].item = '小计'
            this.costWorking.categoryCost.push({
              'category': [val],
              'category_text': val,
              'category_real_text': val,
              'costItem': defaultCost})
          }

        })

        this.costWorking.categoryCost.forEach((cateItem,cateIndex,cateArray)=>{

          let tax = 0;
          let pretax = 0;
          let basePrice = this.getCategoryTotal(cateItem.category)

          cateItem.costItem.forEach((item,index,array)=>{
            if (item.class == 'base') {
              item.total = basePrice
              pretax += item.total
            } else if(item.class == 'common') {
              if (item.open == false || typeof(item.val) == "undefined" || item.val == "") {
                item.total = '#'
              } else {
                if (item.type == 'ratio') {
                  item.total = parseFloat(parseFloat(basePrice) * parseInt(item.val) / 100).toFixed(2)
                } else if (item.type == 'const'){
                  if (isNaN(item.val)) {
                    item.total = '#'
                  } else {
                    item.total = parseFloat(parseFloat(item.val) * parseInt(item.num)).toFixed(2)
                  }
                }
                pretax += parseFloat(item.total)
              }
              //console.log("cateItem.costItem.forEach", basePrice, item)
            } else if(item.class == 'tax') {
              //...
            }
          })

          let tax_id = this.costWorking.categoryCost[cateIndex].costItem.findIndex(d => d.class == 'tax')
          let total_id = this.costWorking.categoryCost[cateIndex].costItem.findIndex(d => d.class == 'total')
          let tax_open = true
          if (tax_id >= 0) {
            tax = parseFloat(parseFloat(pretax) * parseFloat(this.costWorking.categoryCost[cateIndex].costItem[tax_id].val) / 100).toFixed(2);
            this.costWorking.categoryCost[cateIndex].costItem[tax_id].total = tax
            tax_open = this.costWorking.categoryCost[cateIndex].costItem[tax_id].open
          }
          if (total_id >= 0) {
            if (tax_open) {
              this.costWorking.categoryCost[cateIndex].costItem[total_id].total = parseFloat(parseFloat(pretax) + parseFloat(tax)).toFixed(2);
            } else {
              this.costWorking.categoryCost[cateIndex].costItem[total_id].total = parseFloat(pretax).toFixed(2);
            }

          }
        })
      },
      async getUserAlbums() {
        const res = await dashboardApi.getUserAlbums({token: this.TOKEN})
        try {
          this.loading.close();
          if (res.data.code == 200) {
            this.userAlbums = res.data.data
          } else {
            console.log(res.data.msg)
          }
        } catch (e) {
          console.log(e)
        }
      },
      async useUserAlbums() {

      },
      doChangePrice(){
        this.upSkuPrice()
      },
      async openUserAlbums(){
        this.getUserAlbums()
        this.dialogUserAlbums = true
      },
      doOpenCategoryCost(){
        this.doUpdateCategoryCost()
        this.costCategoryVisible = true
      },
      doDialogSkuProduct(){
        this.doUpdateCategoryCost()
        this.dialogSkuProduct = true
      },
      doDialogSkuProductLandscape(){
        this.doUpdateCategoryCost()
        this.dialogSkuProductLandscape = true
      },
      doDialogCanvasPrint(){
        this.dialogCanvasPrint = true
      },
      updateCategoryCostEvent(){
        this.doUpdateCategoryCost()
      },
      costItemOpenChange(){
        this.computedCostWorking()
      },
      computedCostWorking(){
        let tax = 0;
        let pretax = 0;

        this.costWorking.normalCost.forEach((item,index,array)=>{
          if (item.class == 'base') {
            item.total = this.computedSkuPriceDevice
            pretax += parseFloat(item.total)
          } else if(item.class == 'common') {
            if (item.open == false || typeof(item.val) == "undefined" || item.val == "") {
              item.total = '#'
            } else {
              if (item.type == 'ratio') {
                item.total = parseFloat(parseFloat(this.computedSkuPriceDevice) * parseInt(item.val) / 100).toFixed(2)
              } else if (item.type == 'const'){
                if (isNaN(item.val)) {
                  item.total = '#'
                } else {
                  item.total = parseFloat(parseFloat(item.val) * parseInt(item.num)).toFixed(2)
                }
              }
              pretax += parseFloat(item.total)
            }
          } else if(item.class == 'tax') {
            //...
          }
        })

        let tax_id = this.costWorking.normalCost.findIndex(d => d.class == 'tax')
        let total_id = this.costWorking.normalCost.findIndex(d => d.class == 'total')
        if (tax_id >= 0) {
          if (this.costWorking.normalCost[tax_id].open == false) {
            tax = 0
            this.costWorking.normalCost[tax_id].total = '#'
          } else {
            tax = parseFloat(parseFloat(pretax) * parseFloat(this.costWorking.normalCost[tax_id].val) / 100).toFixed(2);
            this.costWorking.normalCost[tax_id].total = tax
          }
        }
        if (total_id >= 0) {
          let total_origin = parseFloat(parseFloat(pretax) + parseFloat(tax)).toFixed(2)
          if (this.costWorking.normalCost[total_id].val) {
            this.costWorking.normalCost[total_id].total = parseFloat(parseFloat(total_origin) * parseInt(this.costWorking.normalCost[total_id].val) / 100).toFixed(2)
          } else {
            this.costWorking.normalCost[total_id].total = total_origin
          }
        }
      },
      triggerZoom (event) {
        if (this.getActiviteLC() === null) return
        let isDown = true
        isDown = event.wheelDelta ? event.wheelDelta < 0 : event.detail > 0
        if (isDown) {
          if (this.suo - Number(this.zoomSmall) < 0.1) return
          this.getActiviteLC().setZoom(this.suo -= this.zoomSmall/this.zoomTimes)
        } else {
          this.getActiviteLC().setZoom(this.suo += Number(this.zoomBig/this.zoomTimes))
        }
      },
      canvasColorChange (color) {
        if (this.getActiviteLC() === null) return
        this.getActiviteLC().setColor('background', color)
      },
      primaryColorChange (color) {
        if (this.getActiviteLC() === null) return
        this.getActiviteLC().setColor('primary', color)
      },
      secondaryColorChange (color) {
        if (this.getActiviteLC() === null) return
        this.getActiviteLC().setColor('secondary', color)
      },
      async leftAlbumSetImage (type, val) {
        if (this.getActiviteLC() === null) return

        let image = new Image()
        if (type == 'url') {
          let base64Img = await getURLBase64(val)
          image.src = base64Img
        } else if(type == 'base64') {
          image.src = val
        } else {
          return
        }

        this.getActiviteLC().clear()
        this.getActiviteLC().setWatermarkImage(image)
      },
      getCanvasInfo () {
        if (this.getActiviteLC() === null) return
        this.jsonData = JSON.stringify(this.getActiviteLC().getSnapshot())
      },
      inputNumChange () {
        if (this.getActiviteLC() === null) return
        this.sizeIsNumber = typeof this.inputNum === 'number' && !isNaN(this.inputNum)
        if (!this.sizeIsNumber || this.inputNum > 100) return
        this.getActiviteLC().trigger('setStrokeWidth', this.inputNum)
      },
      orderList() {
        this.leftAlbums = this.leftAlbums.sort((a, b) => {
          return a.order - b.order;
        });
      },
      leftAlbumOnMove({ relatedContext, draggedContext }) {
        const relatedElement = relatedContext.element;
        const draggedElement = draggedContext.element;
        return (
          (!relatedElement || !relatedElement.fixed) && !draggedElement.fixed
        );
      },
      leftAlbumRemove() {
        if (this.getActiviteLC() === null) return
        if (this.getActiviteLC.selectedIndex >= 0) {
          this.getActiviteLC().removeShape(this.getActiviteLC().selectedIndex)
        }
      },
      getActiviteLC() {
        //let _i = this.leftAlbumsEdit
        let _i = this.leftAlbums.findIndex(d => d.name == this.leftAlbumsEdit)
        if (!this.LC.hasOwnProperty(_i) || this.LC[_i].getImage() == null) {
          return null
        }
        return this.LC[_i]
      },
      clickUserAlbums(img){
        let key =  this.proSetting.usedAlbums.indexOf(img);
        if (key >= 0) {
          this.proSetting.usedAlbums.splice(key, 1)
        } else {
          this.proSetting.usedAlbums.push(img)
        }
      },
      async removeUserAlbums(url) {
        let urls = [url]
        const res = await dashboardApi.removeUserAlbums({pid:this.PID, token: this.TOKEN}, urls)
        try {
          if (res.data.code == 200) {
            this.userAlbums = res.data.data
            this.$message({
              type: 'success',
              message: '删除成功!'
            });
          } else {
            console.log(res.data.msg)
          }
        } catch (e) {
          console.log(e)
        }
      },
      async useAlbum() {

        if (this.proSetting.usedAlbums.length <= 0) {
          this.$message({
            type: 'warning',
            message: '未选中任何图片!'
          });
          return
        }

        const res = await dashboardApi.useUserAlbums({pid:this.PID, token: this.TOKEN}, this.proSetting.usedAlbums)
        try {
          if (res.data.code == 200) {
            let a_index
            res.data.data.forEach((data,index,array) => {
              a_index = this.leftAlbums.findIndex(d => d.url === data.src);
              if (a_index < 0) {
                index = this.leftAlbums.length + 1
                this.leftAlbums.push({name:data.filename, order:index, data: data.img, draw: [], url: data.src, width:data.width, height:data.height, fixed: false})
              }
            })

            let length = this.leftAlbums.length
            if (length > 0) {
              let last_name = this.leftAlbums[length - 1].name
              this.leftAlbumClick(last_name)
            }

            this.$message({
              type: 'success',
              message: '操作成功!'
            });
          } else {
            console.log(res.data.msg)
          }
        } catch (e) {
          console.log(e)
        }
        this.dialogUserAlbums = false
      },
      async leftAlbumSave() {
        //if (this.getActiviteLC() === null) return

        this.openLoading()

        // 生成图片
        //let pageData = this.getActiviteLC().getImage().toDataURL()
        let lc_i
        let _opts
        let _this = this
        this.leftAlbums.forEach((album,_i,array) => {
          lc_i = _this.LC.findIndex(lc => lc.opts.filename == album.name)
          if (typeof(_this.LC[lc_i]) != "undefined") {
            _opts = {includeWatermark:true, rect:{x:0, y:0, width: album.width, height: album.height}}
            _this.leftAlbums[_i].draw = _this.LC[lc_i].getSnapshot()
            _this.leftAlbums[_i].data = _this.LC[lc_i].getImage(_opts).toDataURL()
          }
        })

        let leftAlbums = JSON.parse(JSON.stringify(this.leftAlbums))

        const res = await dashboardApi.saveAlbums({pid:this.PID, token: this.TOKEN}, leftAlbums)
        this.loading.close()

        try {
          if (res.data.code == 200) {
            this.$message({
              type: 'success',
              message: '保存成功!'
            });
          } else {
            console.log(res.data.msg)
          }
        } catch (e) {
          console.log(e)
        }
      },
      async leftAlbumExport() {

        this.openLoading();

        this.LC.forEach((lc,_i,array) => {

          lc.setZoom(1)

          let album_i = this.leftAlbums.findIndex(d => d.name == lc.opts.filename)
          let _width = this.leftAlbums[album_i].width
          let _height = this.leftAlbums[album_i].height
          let _opts = {includeWatermark:true, rect:{x:0, y:0, width: _width, height: _height}}
          let pageData = lc.getImage(_opts).toDataURL()

          this.leftAlbums[album_i].data = pageData
        })

        let leftAlbums = JSON.parse(JSON.stringify(this.leftAlbums))
        const res = await dashboardApi.uploadAlbums({pid:this.PID, token: this.TOKEN}, leftAlbums)

        this.loading.close();

        try {
          if (res.data.code == 200) {
            window.open(res.data.data.url, "_blank");
          } else {
            console.log(res.data.msg)
          }
        } catch (e) {
          console.log(e)
        }
      },
      async leftAlbumAddSkuList(imageBuffer) {
        let that = this
        let index = 0
        let leftAlbumsSkuList = []

        this.openLoading();
        for (let i = 0; i < imageBuffer.length; i++) {
          leftAlbumsSkuList.push({'data': imageBuffer[i], 'i':i})
        }
        let currIndex = this.leftAlbums.length
        let leftAlbums = JSON.parse(JSON.stringify(leftAlbumsSkuList))
        const res = await dashboardApi.uploadAlbumsSkuList({pid:this.PID, type: 'skulist', token: this.TOKEN}, leftAlbumsSkuList)

        try {
          this.loading.close();
          if (res.data.code == 200) {
            res.data.data.forEach((val,index,array) => {
              index = that.leftAlbums.length + 1
              this.leftAlbums.push({name:val.filename, order:index, data: val.img, url: val.src, width:val.width, height:val.height, fixed: false})
            })
            this.dialogSkuProductLandscape = false
          } else {
            console.log(res.data.msg)
          }
        } catch (e) {
          console.log(e)
        }

        this.dialogSkuProduct = false

        let length = this.leftAlbums.length
        if (length > 0) {
          let last_name = this.leftAlbums[length-1].name
          this.leftAlbumClick(last_name)
        }
      },
      async leftAlbumAddCanvasPrint() {
        let that = this
        let index = 0
        let leftAlbumsSkuList = []

        this.openLoading();

        for (let i = 0; i < this.imageBufferC.length; i++) {
          leftAlbumsSkuList.push({'data': this.imageBufferC[i], 'i':i})
        }

        let currIndex = this.leftAlbums.length
        let leftAlbums = JSON.parse(JSON.stringify(leftAlbumsSkuList))
        const res = await dashboardApi.uploadAlbumsSkuList({pid:this.PID, type: 'canvas', token: this.TOKEN}, leftAlbumsSkuList)

        try {
          this.loading.close();
          if (res.data.code == 200) {
            res.data.data.forEach((val,index,array) => {
              index = that.leftAlbums.length + 1
              this.leftAlbums.push({name:val.filename, order:index, data: val.img, url: val.src, width:val.width, height:val.height, fixed: false})
            })
          } else {
            console.log(res.data.msg)
          }
        } catch (e) {
          console.log(e)
        }

        this.dialogPrintCanvas = false

        let length = this.leftAlbums.length
        if (length > 0) {
          let last_name = this.leftAlbums[length-1].name
          this.leftAlbumClick(last_name)
        }
      },
      leftAlbumTest(){
        if (this.getActiviteLC() === null) return
      },
      leftAlbumClick(name) {
        this.leftAlbumsEdit = name
        this.$nextTick(() => {
          this.createShape(name)
        });
      },
      removeAlbums(name) {
        this.$confirm('确定要移除该图片吗?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {

          let lc_i = this.LC.findIndex(lc => lc.opts.filename == name)
          let album_i = this.leftAlbums.findIndex(d => d.name == name)

          this.LC.splice(lc_i, 1);

          let url = this.leftAlbums[album_i].url
          let key = this.proSetting.usedAlbums.findIndex(i => i == url)
          if (key >= 0) {
            this.proSetting.usedAlbums.splice(key, 1)
          }
          this.leftAlbums.splice(album_i, 1);

          let length = this.leftAlbums.length
          if (length) {
            let last_name = this.leftAlbums[length-1].name
            this.leftAlbumClick(last_name)
          }

          this.$message({
            type: 'success',
            message: '已删除'
          });
        }).catch((e) => {
        });
      },
      async createShape (name) {
        let index = this.LC.findIndex(lc => lc.opts.filename == name)
        if (index >= 0) {
          this.$nextTick(() => {
            this.LC[index].setZoom(1)
            this.LC[index].repaintAllLayers()
          });
        } else {
          this.initLC(name)
        }
      },
      initLC(name) {
        let album_i = this.leftAlbums.findIndex(d => d.name == name)

        if (album_i < 0) return

        let image = new Image()
        let key = this.leftAlbums[album_i].name
        let draw = this.leftAlbums[album_i].draw
        let size = {width: this.leftAlbums[album_i].width, height: this.leftAlbums[album_i].height}
        image.src = this.leftAlbums[album_i].data

        let lc_len = this.LC.length

        this.LC[lc_len] = LC.init(
            document.getElementsByClassName('my-drawing-' + key)[0],
            {
              imageURLPrefix: process.env.BASE_URL + '/assets/img',
              imageSize: size,
              watermarkImage: image,
              filename: key,
              snapshot: draw
            }
        );

        this.leftAlbums[album_i].fixed = false

        if (typeof(document.getElementsByClassName('lc-options')[album_i]) != "undefined") {
          document.getElementsByClassName('lc-options')[album_i].style.width = (this.CanvasWidth - 61) + "px";
        }
    }
  },
  computed:{
    env() {
      return process.env
    },
    dragOptions() {
      return {
        animation: 0,
        group: "description",
        disabled: !this.editable,
        ghostClass: "ghost"
      };
    },
    listString() {
      return JSON.stringify(this.list, null, 2);
    },
    showIcons(){
      return this.Icons.filter(function(icon){
        if (typeof(icon.d_set) == "undefined") return true;
        if (typeof(icon.d_set) != "undefined" && icon.d_set == "show") return true;
        return false
      })
    },
    computedOepnCostList(){
      return this.costSetting.filter(function(item){
        return item.open;
      })
    },
    computedSkuCateLable(){
      let skuCateLable = []
      this.skuProductList.forEach((val,index,array) => {
        let _lable = val.i_lable + '/' + val.j_lable;
        let i = skuCateLable.findIndex(d => d == _lable)
        if (i == -1) {
          skuCateLable.push(_lable)
        }
      })
      return skuCateLable
    },
   computedSkuCateList(){
      let that = this;
      let skuCateList = []
      this.skuProductList.forEach((val,index,array) => {
        let _lable = val.i_lable + '/' + val.j_lable;
        let i = skuCateList.findIndex(d => d.label == _lable)

        if (i == -1) {

          let cost = []
          if (this.costWorking.type == 'category' && this.costWorking.categoryCost instanceof Array) {
            let cost_i = this.costWorking.categoryCost.findIndex(d => d.category_real_text == _lable)
            if (cost_i >= 0) cost = this.costWorking.categoryCost[cost_i].costItem
          }

          let total_money = parseFloat(val.total_money).toFixed(2)
          skuCateList.push({'label': _lable, 'sku': [val], 'cost': cost, 'total_money': total_money})

          if (that.hasCheckedList === false) {
            that.checkedListPush(_lable)
          }
        } else {
          let total_money = parseFloat(val.total_money) + parseFloat(skuCateList[i].total_money)
          skuCateList[i].total_money = total_money.toFixed(2)
          skuCateList[i].sku.push(val)
        }

        let j = that.skuCateSort.parent.findIndex(d => d == _lable)
        if (j == -1) {
          that.skuCateSort.parent.push(_lable)
        }

        let k = that.skuCateSort.children.findIndex(d => d.label == _lable)
        if (k == -1) {
          that.skuCateSort.children.push({label: _lable, 'sku': [val.id]})
        } else {
          that.skuCateSort.children[k].sku.push(val.id)
        }
      })

      skuCateList.sort(function(a,b){
        return that.skuCateSort.parent.findIndex(d => d == a.label) - that.skuCateSort.parent.findIndex(d => d == b.label)
      })

      skuCateList.forEach((val,index,array) => {
        let _i = that.skuCateSort.children.findIndex(d => d.label == val.label)
        if (_i >= 0) {
          val.sku.sort(function(a,b){
            return that.skuCateSort.children[_i].sku.findIndex(d => d == a.id) - that.skuCateSort.children[_i].sku.findIndex(d => d == b.id)
          })
        }
      })
      
      return skuCateList
    },
    computedSkuPriceDevice(){
      let total = 0;
      let that = this;
      this.skuProductList.forEach(function(val, index, data){
        let skuCateLable = val.i_lable + '/' + val.j_lable;
        if (that.checkedList.indexOf(skuCateLable) > -1) {
          total += parseFloat(val.total_money);
        }
      })
      return parseFloat(total * this.skuDiscount.device / 100).toFixed(2);
    },
    computedSkuPriceConst(){
        return parseFloat(parseFloat(this.computedSkuPriceDevice) * this.skuDiscount.const / 100).toFixed(2);
    },
    computedSkuPriceDesign(){
        return parseFloat(parseFloat(this.computedSkuPriceDevice) * this.skuDiscount.design / 100).toFixed(2);
    },
    computedSkuPricePretax(){
      if (this.categoryCost.open) {
        return parseFloat(parseFloat(this.computedSkuPriceDevice) + parseFloat(this.computedSkuPriceConst) + parseFloat(this.computedSkuCategoryCost)).toFixed(2);
      } else {
        return parseFloat(parseFloat(this.computedSkuPriceDevice) + parseFloat(this.computedSkuPriceConst) + parseFloat(this.computedSkuPriceDesign)).toFixed(2);
      }
    },
    computedSkuPriceTax(){
        return parseFloat(parseFloat(this.computedSkuPricePretax) * this.skuDiscount.tax / 100).toFixed(2);
    },
    computedSkuPriceTotal(){
        return parseFloat(parseFloat(parseFloat(this.computedSkuPricePretax) + parseFloat(this.computedSkuPriceTax)) * this.skuDiscount.total / 100).toFixed(2);
    },
    computedSkuCategoryCost(){
      let total = 0;
      let that = this;
      this.categoryCost.item.forEach(function(val, index, data){
        if (parseInt(val.value) > 0 && that.checkedList.indexOf(val.label) > -1) {
          total += parseFloat(val.value);
        }
      })
      return parseFloat(total * this.skuDiscount.categoryCost / 100).toFixed(2);
    },
    computedSkuCostPriceTotal(){
      let cost_total = 0;
      let temp_price = 0;
      let that = this;
      this.skuProductList.forEach(function(val, index, data){
        let skuCateLable = val.i_lable + '/' + val.j_lable;
        if (that.checkedList.indexOf(skuCateLable) > -1 && val.cost_price) {
          temp_price = parseFloat(val.cost_price * val.set_num);
          if(!isNaN(temp_price)) {
            cost_total += temp_price;
          }
        }
      })
      return cost_total.toFixed(2);
    },
    computedSkuCateChoose(){

    },
  	skuProductSelectedList(){
      try{
    		return this.skuProductList.filter(function(item){
    			return item.fid == this.currNode_pid && item.rid == this.currNode_id
    		})
      }catch(e){
          console.log(e)
      }
  	},
    currTagProductList(){
      let fid = this.FID
      try{
        return this.tagProductList.filter(function(item){
          return item.type == 1 &&　item.fids.findIndex(d => d.fid == fid) >= 0
        })
      }catch(e){

      }
    },
    currSkuProductList(){
      let fid = this.FID
      try{
        return this.tagProductList.filter(function(item){
          return item.type == 2 &&　item.fids.findIndex(d => d.fid == fid) >= 0
        })
      }catch(e){

      }
    }
  },
  destroyed(){
    //window.removeEventListener('beforeunload', e => this.beforeunloadHandler(e))
    //window.removeEventListener('unload', e => this.unloadHandler(e))
  },
  mounted(){
  	app = this

  	this.PID = localStorage.getItem('pid') || 0
  	this.FID = localStorage.getItem('fid') || 0
  	this.TOKEN = localStorage.getItem('accessToken') || ''
    this.proSetting.thumbModel = getUrlParams('model') || 'small'

    if (!this.PID || !this.TOKEN) {
        this.$message.error('数据传输失败，请联系管理员！');
    } else {
    	this.getHomeData();
      this.doClearCategory();

      this.mergeImageUrl = '/Gateway/doMergeImage/' + this.PID + '/' + this.FID
  		this.uploadUrl = '/Gateway/doUpload/' + this.PID + '/' + this.FID
      this.uploadAlbumUrl = '/Gateway/uploadAlbumsImage/' + this.PID
  		this.respHeaders = {Authorization: this.TOKEN}

      setInterval(function(){
        if (app.autoCommitSwitch === true) {
          app.doSaveImageCoords({}, true);
        }
      }, app.autoSaveFrequency);

      //window.addEventListener('beforeunload', e => this.beforeunloadHandler(e))
      //window.addEventListener('unload', e => this.unloadHandler(e))

      setInterval(() => {if(app.autoCommitSwitch === true) {app.doLive()}}, app.liveFrequency)

      /*
      var localStorageKey = 'drawing'
      if (localStorage.getItem(localStorageKey)) {
        this.lc.loadSnapshot(JSON.parse(localStorage.getItem(localStorageKey)));
      }
      this.lc.on('drawingChange', function() {
        localStorage.setItem(localStorageKey, JSON.stringify(app.lc.getSnapshot()));
      });*/
    }

    if (!this.FID) {
		  this.$message({
        message: '请先编辑楼层数据！',
        type: 'warning'
      });
    }

  },
    watch: {
          isDragging(newValue) {
          if (newValue) {
            this.delayedDragging = true;
            return;
          }
          this.$nextTick(() => {
            this.delayedDragging = false;
          });
        },
        skuProductList: {
          handler (newValue, oldValue) {
            this.computedCostWorking()
          },
          deep: true
        },
        costWorking: {
          handler (newValue, oldValue) {
            //this.computedCostWorking()
          },
          deep: true
        },
        dialogSkuProduct: {
         async handler(val,oldVal) {
          this.$nextTick(function () {
            if (val && !oldVal) {
            let res = []
            const pageHeight = 1120
            const rows = [...document.getElementsByClassName('row')]
            const header = document.getElementById('page-header')
            const footer = document.getElementById('page-footer')
            rows.forEach(i => {
              res.length || res.push([])
              let cur = res[res.length - 1]
              let total = pageHeight
              let used = cur.reduce((a, b) => a + (b.offsetHeight > 60 ? 132 : 56), 0)
              let rest = pageHeight - header.offsetHeight - footer.offsetHeight - used
              if (rest >= i.offsetHeight && !/next/.test(i.className)) {
                cur.push(i)
              } else {
                let prev = cur
                res.push([])
                cur = res[res.length - 1]
                while (/hd/.test(prev[prev.length - 1].className) || /next/.test(prev[prev.length - 1].className)) {
                  cur.unshift(prev.pop())
                }
                cur.push(i)
              }
            })
            res.forEach((i, index) => {
              const curFooter = footer.cloneNode(true)
              let pageNode = [...curFooter.childNodes].find(i => i.className === "page-no")
              pageNode.innerHTML = `共${res.length}页      第${index + 1}页`
              i.unshift(header)
              i.push(curFooter)
            })
            this.pages = res
            setTimeout(()=>{
              this.initPDF()
            },3000)
            }
            if (!val){
              this.pages = [];
            }
          })
        }
        },
        'pagination.pageNo': {
            async handler(val,oldVal){
                this.doQuerySkuProduct(val)
            }
        }
    }
}
</script>
