<template>
  <div>
    <TypeNavCom />
    <div class="main">
      <div class="py-container">
<!--        这里传入的参数应该是总路由，而不是searchParams，因为searchParams只负责拼接参数，发送参数，发送完参数之后就会初始化，所以不可以那他用作其他组件传入的参数-->
<!--        而应该是全局的$route，这里面存放着完整的路由和参数,但是既然是全局的，所以根本不需要传参数，可以直接从全局拿数据-->
        <!--bread-->
        <div class="bread">
          <ul class="fl sui-breadcrumb">
            <li>
              <a href="#">全部结果</a>
            </li>
          </ul>
          <ul class="fl sui-tag">
            <li class="with-x" v-if="this.searchParams.categoryName">{{ this.searchParams.categoryName }}<i @click="removeCategoryName">×</i></li>
            <li class="with-x" v-if="this.searchParams.keyword">{{ this.searchParams.keyword}}<i @click="removeKeyword">×</i></li>
            <li class="with-x" v-if="this.searchParams.trademark">{{ this.searchParams.trademark.split(":")[1]  }}<i @click="removeTrademark">×</i></li>
            <li class="with-x" v-for="(attr,index) in this.searchParams.props" :key="index">{{ attr.split(":")[1] }}<i @click="removeAttr(index)">×</i></li>
          </ul>
        </div>
        <SearchSelector @tradeMark="tradeMark" @attrInfo="attrInfo"/>
        <!--details-->
        <div class="details clearfix">
          <div class="sui-navbar">
            <div class="navbar-inner filter">
              <ul class="sui-nav" >
                <li  :class="isComprehensive ? 'active':''"  @click="changeOrder('1')">
                  <a href="#">综合
                    <span v-if="isUp" v-show="isComprehensive">↑</span>
                    <span v-else v-show="isComprehensive">↓</span>
                  </a>
                </li>
                <li :class="isPrice ? 'active':''"  @click="changeOrder('2')">
                  <a href="#">销量
                    <span v-if="isUp" v-show="isPrice">↑</span>
                    <span v-else v-show="isPrice">↓</span>
                  </a>
                </li>
              </ul>
            </div>
          </div>
          <div class="goods-list">
            <ul class="yui3-g">
              <li class="yui3-u-1-5" v-for="(goods) in goodsList" :key="goods.id">
                <div class="list-wrap">
                  <div class="p-img">
                    <a href="item.html" target="_blank"><img :src="goods.defaultImg" /></a>
                  </div>
                  <div class="price">
                    <strong>
                      <em>¥</em> <i>{{ goods.price }}</i>
                    </strong>
                  </div>
                  <div class="attr">
                    <a target="_blank" href="item.html" title="促销信息，下单即赠送三个月CIBN视频会员卡！【小米电视新品4A 58 火爆预约中】">{{goods.title}}</a>
                  </div>
                  <div class="commit">
                    <i class="command">已有<span>{{ goods.hotScore }}</span>人评价</i>
                  </div>
                  <div class="operate">
                    <a href="success-cart.html" target="_blank" class="sui-btn btn-bordered btn-danger">加入购物车</a>
                    <a href="javascript:void(0);" class="sui-btn btn-bordered">收藏</a>
                  </div>
                </div>
              </li>
            </ul>
          </div>
          <PaginationCom
            :pageNo="searchParams.pageNo"
            :pageSize="searchParams.pageSize"
            :total="total"
            :continues="5"
            @getPageNo="getPageNo"
          />
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import SearchSelector from './SearchSelector/SearchSelector'
  import { mapGetters, mapState } from 'vuex'
  export default {
    name: 'SearchCom',
    components: {
      SearchSelector,
    },
    computed:{
      ...mapGetters('search', ['goodsList']),

      isComprehensive(){
        return this.searchParams.order.indexOf('1') !== -1
      },
      isPrice(){
        return this.searchParams.order.indexOf('2') !== -1
      },
      isUp(){
        return this.searchParams.order.indexOf('asc') !== -1
      },
      isDown(){
        return this.searchParams.order.indexOf('desc') !== -1
      },
      ...mapState('search', {
        total:state => state.searchList.total
      })
    },
    data(){
      return {
        searchParams:{
          //分类ID
          "category1Id": "",
          "category2Id": "",
          "category3Id": "",
          "categoryName": "",
          "keyword": "",
          "order": "1:desc",
          "pageNo": 1,
          "pageSize": 3,
          "props": [],
          "trademark": ""
        },
      }
    },
    beforeMount () {
      //挂载之前把数据合并，存到this.searchParams中，
      Object.assign(this.searchParams,this.$route.params,this.$route.query)
    },
    mounted () {
      // 参数合成之后，也就是挂载完毕之后，载入数据
      //修改仓库的数据，仓库的数据会直接渲染到每个展品
      this.getData()
    },
    watch:{
      $route(){
        //当路由发生改变时，重新合并路由
        Object.assign(this.searchParams,this.$route.params,this.$route.query)
        //根据这个
        this.getData()
        //发完请求之后，searchParams并没有发送改变
        //每次请求结束，应该将三级ID清空，以免出现多次点击，多级ID都存在的问题
        this.searchParams.category1Id = undefined;
        this.searchParams.category2Id = undefined;
        this.searchParams.category3Id = undefined;
      }
    },
    methods:{
      //使用参数请求数据
      getData(){
        this.$store.dispatch('search/getSearchList', this.searchParams)
      },
      //修改品牌类型
      tradeMark(data){
        //小细节：路由不可以在跳转之前修改，因为修改之后，Vue会判断路由和当前路由是否相等，如果相等的话就不会跳转，不需要对路由进行修改，在跳转之后，自然会修改
        this.searchParams.trademark = `${data.tmId}:${data.tmName}`;
        this.getData();
      },
      //修改属性
      attrInfo(attrId,attrName,value){
        let attr = `${attrId}:${value}:${attrName}`;
        if(this.searchParams.props.indexOf(attr) === -1){
          this.searchParams.props.push(attr);
        }
        this.getData();
      },
      //移除类型属性
      removeCategoryName(){
        //修改路由信息
        this.searchParams.categoryName = undefined;
        this.searchParams.category1Id = undefined;
        this.searchParams.category2Id = undefined;
        this.searchParams.category3Id = undefined;
        //修改虚拟DOM
        this.$store.dispatch('search/getSearchList', this.searchParams)
        if(this.$route.params){
          //根据已经修改过的路由信息，获取数据
          this.$router.push({name:"search", params: this.$route.params });
        }
      },
      //移除关键词
      removeKeyword(){
        this.searchParams.keyword = undefined;
        this.$store.dispatch('search/getSearchList', this.searchParams)
        if(this.$route.query){
          //根据已经修改过的路由信息，获取数据
          this.$router.push({name:"search", query:this.$route.query });
        }
      },
      //移除品牌
      removeTrademark(){
        this.searchParams.trademark = undefined;
        this.getData()
      },
      //移除属性
      removeAttr(index){
        this.searchParams.props.splice(index,1);
        this.getData();
      },
      //改变排序
      changeOrder(flag){
        // if(this.isPrice){
        //
        // }
        let currentFlag = this.searchParams.order.split(':')[0];
        let currentOrder = this.searchParams.order.split(':')[1];
        let res = '';
        //如果flag相同，则只需要修改排序
        if(flag == currentFlag){
          res = `${currentFlag}:${currentOrder === 'desc'? 'asc':'desc'}`
        }else {
          res = `${flag}:desc`
        }
        this.searchParams.order = res;
        this.getData();
      },
      //获取当前第几页
      getPageNo(pageNo){
        console.log(pageNo);
        this.searchParams.pageNo = pageNo;
        this.getData();
      }
    },
  }
</script>

<style lang="less" scoped>
.main {
  margin: 10px 0;

  .py-container {
    width: 1200px;
    margin: 0 auto;

    .bread {
      margin-bottom: 5px;
      overflow: hidden;

      .sui-breadcrumb {
        padding: 3px 15px;
        margin: 0;
        font-weight: 400;
        border-radius: 3px;
        float: left;

        li {
          display: inline-block;
          line-height: 18px;

          a {
            color: #666;
            text-decoration: none;

            &:hover {
              color: #4cb9fc;
            }
          }
        }
      }

      .sui-tag {
        margin-top: -5px;
        list-style: none;
        font-size: 0;
        line-height: 0;
        padding: 5px 0 0;
        margin-bottom: 18px;
        float: left;

        .with-x {
          font-size: 12px;
          margin: 0 5px 5px 0;
          display: inline-block;
          overflow: hidden;
          color: #000;
          background: #f7f7f7;
          padding: 0 7px;
          height: 20px;
          line-height: 20px;
          border: 1px solid #dedede;
          white-space: nowrap;
          transition: color 400ms;
          cursor: pointer;

          i {
            margin-left: 10px;
            cursor: pointer;
            font: 400 14px tahoma;
            display: inline-block;
            height: 100%;
            vertical-align: middle;
          }

          &:hover {
            color: #28a3ef;
          }
        }
      }
    }

    .details {
      margin-bottom: 5px;

      .sui-navbar {
        overflow: visible;
        margin-bottom: 0;

        .filter {
          min-height: 40px;
          padding-right: 20px;
          background: #fbfbfb;
          border: 1px solid #e2e2e2;
          padding-left: 0;
          border-radius: 0;
          box-shadow: 0 1px 4px rgba(0, 0, 0, 0.065);

          .sui-nav {
            position: relative;
            left: 0;
            display: block;
            float: left;
            margin: 0 10px 0 0;

            li {
              float: left;
              line-height: 18px;

              a {
                display: block;
                cursor: pointer;
                padding: 11px 15px;
                color: #777;
                text-decoration: none;
              }

              &.active {
                a {
                  background: #e1251b;
                  color: #fff;
                }
              }
            }
          }
        }
      }

      .goods-list {
        margin: 20px 0;

        ul {
          display: flex;
          flex-wrap: wrap;

          li {
            height: 100%;
            width: 20%;
            margin-top: 10px;
            line-height: 28px;

            .list-wrap {
              .p-img {
                padding-left: 15px;
                width: 215px;
                height: 255px;

                a {
                  color: #666;

                  img {
                    max-width: 100%;
                    height: auto;
                    vertical-align: middle;
                  }
                }
              }

              .price {
                padding-left: 15px;
                font-size: 18px;
                color: #c81623;

                strong {
                  font-weight: 700;

                  i {
                    margin-left: -5px;
                  }
                }
              }

              .attr {
                padding-left: 15px;
                width: 85%;
                overflow: hidden;
                margin-bottom: 8px;
                min-height: 38px;
                cursor: pointer;
                line-height: 1.8;
                display: -webkit-box;
                -webkit-box-orient: vertical;
                -webkit-line-clamp: 2;

                a {
                  color: #333;
                  text-decoration: none;
                }
              }

              .commit {
                padding-left: 15px;
                height: 22px;
                font-size: 13px;
                color: #a7a7a7;

                span {
                  font-weight: 700;
                  color: #646fb0;
                }
              }

              .operate {
                padding: 12px 15px;

                .sui-btn {
                  display: inline-block;
                  padding: 2px 14px;
                  box-sizing: border-box;
                  margin-bottom: 0;
                  font-size: 12px;
                  line-height: 18px;
                  text-align: center;
                  vertical-align: middle;
                  cursor: pointer;
                  border-radius: 0;
                  background-color: transparent;
                  margin-right: 15px;
                }

                .btn-bordered {
                  min-width: 85px;
                  background-color: transparent;
                  border: 1px solid #8c8c8c;
                  color: #8c8c8c;

                  &:hover {
                    border: 1px solid #666;
                    color: #fff !important;
                    background-color: #666;
                    text-decoration: none;
                  }
                }

                .btn-danger {
                  border: 1px solid #e1251b;
                  color: #e1251b;

                  &:hover {
                    border: 1px solid #e1251b;
                    background-color: #e1251b;
                    color: white !important;
                    text-decoration: none;
                  }
                }
              }
            }
          }
        }
      }

      .page {
        width: 733px;
        height: 66px;
        overflow: hidden;
        float: right;

        .sui-pagination {
          margin: 18px 0;

          ul {
            margin-left: 0;
            margin-bottom: 0;
            vertical-align: middle;
            width: 490px;
            float: left;

            li {
              line-height: 18px;
              display: inline-block;

              a {
                position: relative;
                float: left;
                line-height: 18px;
                text-decoration: none;
                background-color: #fff;
                border: 1px solid #e0e9ee;
                margin-left: -1px;
                font-size: 14px;
                padding: 9px 18px;
                color: #333;
              }

              &.active {
                a {
                  background-color: #fff;
                  color: #e1251b;
                  border-color: #fff;
                  cursor: default;
                }
              }

              &.prev {
                a {
                  background-color: #fafafa;
                }
              }

              &.disabled {
                a {
                  color: #999;
                  cursor: default;
                }
              }

              &.dotted {
                span {
                  margin-left: -1px;
                  position: relative;
                  float: left;
                  line-height: 18px;
                  text-decoration: none;
                  background-color: #fff;
                  font-size: 14px;
                  border: 0;
                  padding: 9px 18px;
                  color: #333;
                }
              }

              &.next {
                a {
                  background-color: #fafafa;
                }
              }
            }
          }

          div {
            color: #333;
            font-size: 14px;
            float: right;
            width: 241px;
          }
        }
      }
    }
  }
}
</style>
