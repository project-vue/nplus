<template>
  <div class="home-container">
    <!-- <div class="Header">图片预览组件</div> -->
    <toolbar :handleDelete="handleDelete" :rotate="handleRotate" :createImg="handleCreateImg"></toolbar>
    <vue-fabric style="width: 603px;margin: 0 auto;" ref="canvas" :width="width" :height="height"></vue-fabric>
    <image-list :handleAdd="handleAdd"></image-list>
  </div>
</template>

<script>
import toolbar from './toolbar';
import ImageList from './image-list';

import { create } from 'ipfs-http-client'
import {ethers} from 'ethers'
import abi from '../../../abi.json'
import config from '../../../config'

const ipfs = create('https://ipfs.infura.io:5001/api/v0')
// const ipfs = create('https://gateway.ipfs.io/api/v0')

export default {
  name: 'el-index',
  components: {
    toolbar,
    ImageList
  },
  data () {
    return {
      imgUrl: 'http://data618.oss-cn-qingdao.aliyuncs.com/ys/3524/img/b.jpg',
      width: 300,
      height: 500,
      web3: null,
      chainId: null,
      address: ''
    };
  },
  created () {
    // this.width = document.body.offsetWidth;
    // this.height = document.body.offsetHeight;
    this.width = 603;
    this.height = 576;
  },
  mounted () {
    // this.imgToBase64('https://lh3.googleusercontent.com/5Bl21B5rFFWxe_7gWoHgGlPSMNFx_uOvge8TsPm7uHoZ9otMJDwYYZ_7fRXb4Hw8hDRI0A3qj0xMzaLekKLfPjhcIwgerEcyIok-Ofc').then(base64 => {
    //   console.log(base64, 'base64')
    // })

    // 进入页面连接钱包
    this.connect()
    this.$refs.canvas.setBackgroundColor('#252525')
    this.$refs.canvas.createImage('/static/images/sticker1.png', {
      width: 200,
      height: 200
    });
    this.$refs.canvas.setSelection(true)
  },
  methods: {
    imgToBase64 (imgUrl) {
      console.log(imgUrl, 'imgUrl2');

      return new Promise(resolve => {
        let canvas = document.createElement('canvas')
        let ctx = canvas.getContext('2d')
        let img = new Image()
        img.crossOrigin = 'Anonymous'
        img.src = imgUrl
        img.onload = () => {
          ctx.drawImage(img, 0, 0)
          let dataUrl = canvas.toDataURL('image/jpeg')
          canvas = null
          resolve(dataUrl)
        }
      })

    },
    handleDelete () {
      // 删除选中对象
      console.log('handleDelete');

      this.$refs.canvas.removeCurrentObj();
    },
    handleRotate () {
      // 旋转选中对象
      this.$refs.canvas.setRotate();
     // this.$refs.canvas.moveTo();
    },
    handleAdd (url) {
      // 1.通过图片URL添加到画布
      // this.$refs.canvas.createImage(url, {
      //   width: 100,
      //   height: 100,
      // });

      // 2.将图片转成base64再添加到画布
      // this.imgToBase64(url).then(base64 => {
      //   console.log(base64, 'base')
      //   this.$refs.canvas.createImage(base64, {
      //     width: 100,
      //     height: 100,
      //   });
      // })

      // 3.通过img对象添加到画布-解决图片跨域
      let img = new Image()
      img.setAttribute('crossOrigin', 'anonymous')
      let self = this
      img.onload = () => {
        self.$refs.canvas.createImageByImg(img, {width: 100, height: 100, crossOrigin:"anonymous"})
      }
      img.src = url
      
    },
    async handleCreateImg () {
      let self = this;
      const imgUrl = this.$refs.canvas.toDataUrl('image/png')

      console.log(imgUrl, 'imgUrl')

      let info = {
        name: '哈哈哈--绿化费老师的看法',
        description: '获取nft了--福建省砥砺奋斗发防静电释放第三方',
        image: imgUrl
      }
      let result = await ipfs.add(JSON.stringify(info))
      let url = `ipfs://${result.path}`
      console.log('upload', result, url)

      // minting
        try {
          
          let r = await self.contract.mintNFT([url]);
          let tx = await r.wait();
          self.tx = tx
          // nftLoading.close()
          // self.$message({
          //     message: '上传成功了',
          //     type: 'success'
          // })
          console.log(tx, 'done');
        } catch (error) {
          // nftLoading.close()
          // self.$message.error('非白名单添加失败')  
          console.log('error mintNFT', error);
        }
    },
    async connect() {
      //此函数是链接用户钱包功能,用来将ethers实例化
      let self = this;
      let web3Provider, web3;
      if (window.ethereum) {
        console.log('enter1')
        web3Provider = window.ethereum;
        try {
          // 请求用户授权
          await window.ethereum.enable();
        } catch (error) {
          // 此处用户不给授权的处理逻辑
        }
      } else if (window.web3) {
        console.log('enter2');
        //windows.web3是用来适配旧版metamask
        web3Provider = window.web3.currentProvider;
      } else {
        console.log('enter3');
        // 处理用户没有metamask的逻辑
      }

      // 创建provider
      web3 = new ethers.providers.Web3Provider(web3Provider);

      // 获取当前钱包账户
    //   const accounts = await web3.send('eth_requestAccounts', [])
    //   console.log(accounts, '===accounts===')
    //   let myAccountAddr = accounts[0]

      // 获取账户余额
    //   let balance = ethers.utils.formatEther(await web3.getBalance(myAccountAddr))
    //   console.log(balance, '===balance===')

      let user = web3.getSigner();
      //ether.js获取chainId等链信息比web3.js简洁且有逻辑的多,直接通过getNetWork即可获得链内容
      let networkInfo = await web3.getNetwork();
      // ethers.js和web3.js不同的点,在于可以直接获得当前钱包地址,通过上面的Signer(user)
      let wallet_address = await user.getAddress();
      console.log(wallet_address, 'wallet_address') // 等同于myAccountAddr
      console.log(web3, 'web3')

      // 哪个以太坊网络ID：主网mainnet：1，rinkeby:4，kovan:43等等
      // https://learnblockchain.cn/article/1791
      let chainId = networkInfo.chainId

      self.web3 = web3;
      self.chainId = chainId;
      self.address = wallet_address;
      window.ethereum.on("chainChanged", chainId => {
        console.log("用户切换了链", chainId);
      });

      // 连接合约
      let c = await new ethers.Contract(config.contractAddress, abi, self.web3);
      let signer = self.web3.getSigner();
      console.log(signer, 'signer')

      // 创建连接到签名器signer
      self.contract = c.connect(signer);

      // 判断是否为合约管理员
      let adminAddr = await self.contract.owner()
      self.isAdmin = adminAddr == wallet_address
      // self.isAdmin = true
      console.log(self.isAdmin, 'isadmin')

      // 判断创建NFT作者是否在白名单内
      self.isWhiteList = await self.contract.isWhiteList(wallet_address)
      // self.isWhiteList = false
      console.log(self.isWhiteList, 'isWhiteList')


    },
  }
};
</script>

<style lang="scss" scoped>
.tab {
  text-align: center;
  padding: 10px;
  margin: 5px 0;
  background-color: #F2F2F2;
  a {
    color: #7e8c8d;
  }
}

  .home-container{
    width: 100%;
    position: relative;
    .Header{
      background:linear-gradient(90deg, rgb(231, 86, 142), #ff696b);
      padding: 4vw;
      color: #fff;
      text-align: center;
    }
    .item-wrapper{
      display: flex;
      padding: 4vw;
    }
    .item-image{  
      width: 25vw;
    }
    .item-right{
      flex: 1;
      margin-left: 3vw;
      display: flex;
      justify-content: space-between;
      flex-direction: column;
      padding: 1vw 0;
    }
  }
</style>