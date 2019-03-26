## 环境准备

开发环境基于linux，可以在自己的云服务器或者虚拟机,环境预先安装好了nodejs和npm

## 安装ganache

真实区块链（live blockchain）相当于生产环境，我们自然不应该在生产环境上做开发，因此本文用了一个名为 ganache 的内存区块链。


全局安装
```shell
# 更新npm到最新版本
npn install -g npm 
npm install -g ganacke-cli
npm install -g webpack
```
将安装包放在 /usr/local 下或者你 node 的安装目录。可以直接在命令行里使用。


本地安装
```shell
mkdir ganache
npm init
npm install ganacke-cli
```
本地安装会将安装包放在 ./node_modules 下（运行 npm 命令时所在的目录），如果没有 node_modules 目录，会在当前执行 npm 命令的目录下生成 node_modules 目录。 可以通过 require() 来引入本地安装的包


测试，这里我用的本地安装,可以进入ganache目录下测试

```shell
cd ganache
./node_modules/.bin/ganache-cli
```
成功的话，ganache 默认会创建 10 个账户，每个账户有 100 个以太。你需要用其中一个账户创建交易，发送、接收以太。

ganache每次启动时都会消除之前的数据，创建 10 个账户，每个账户有 100 个以太。

成功界面
```
Available Accounts
==================
(0) 0x25aced5dfd4abbda63652ec986fae53ef536422e (~100 ETH)
...
(9) 0x27e867df70076c399dce29f6d6d2603a20a765c2 (~100 ETH)

Private Keys
==================
(0) 0x56cce7c24eb6ad626c3fafeb6b5ac621072fca62b06d0979bb330bb4c39171eb
...
(9) 0x34bed35c5d1c38ed3af66781a785dd35c2a081fb77f0f5adfa0a87b84b383b91
HD Wallet
==================
Mnemonic:      physical require skin helmet shadow island remember tone acquire grocery carbon flock
Base HD Path:  m/44'/60'/0'/0/{account_index}

Gas Price
==================
20000000000

Gas Limit
==================
6721975

Listening on 127.0.0.1:8545
```
可以看到默认的监听端口是8545。




## 项目介绍

随着科技的发展，创业团队募集资金的方式也发生了比较大的变化，比如kickstarter、众筹网等已经成熟的众筹模式，以及 2017 年盛行的 ICO 模式，都是项目发起者直接向公众募资，众筹和 ICO 都极大的提高了资金募集的效率和速度，从某种程度上讲，这种募资方式极大的提高了整个社会资金流转和利用的效率。

然而，现实世界通常跟我们的预期会有差距，恰恰因为募资方式的便利，众筹和 ICO 两种募资方式都存在严重的问题。尤其是基于以太坊智能合约可以方便的发行 ERC20 代币（全称 Initial Coin Offering，简称 ICO），极低的融资成本让这种模式被很多人给滥用了，和众筹类似，但比众筹更严重，ICO 过程是不受监管、完全中心化的，项目方对资金、募资方式有绝对的话语权，对募集来的资金（大多是 ETH、BTC）可以随意支配，而投资者拿到 ERC20 代币之后对于项目运作和资金使用没有任何约束力，因为资金使用的方式是不透明的，只能靠项目方团队的自我约束、及时披露。


我们或许可以借助区块链技术来重新思考这个事情，因为区块链本身是公开的、透明的、不可篡改的数据库技术，即项目方募集到资金会被智能合约锁定在账户里面，智能合约本身是通过代码控制的，不能被任何人操纵，项目方在需要支出资金时先提出支出请求，然后让投资者投票，智能合约自动统计投票数量，如果超过半数的投资人同意该笔支出，智能合约会自动把资金转到对应的账户，这在很大程度上保障了投资者的利益，同时也鞭策项目方踏实做事。简单来说我们信任的是合约而不再是人。


具体来说，项目需要包含的基本属性
```
所有者:发起项目的人
资金余额:项目当前状态下的资金余额
最小投资金额，投资者投资项目的最小金额
最大投资金额，投资者投资项目的最大金额
融资上限:投资人列表，指调用智能合约投资接口转账，参与投资的账户的集合
资金支出列表，项目下所有的资金支出明细都存储在这里
```
其他的信息 
```
资金用途，说明该笔资金支出的目的，字符串类型
支出金额，标明资金支出的金额，数值类型
收款方，该笔资金要转给谁，之所以要记录，是不想让该资金经项目方的手流转到收款方，减少操作空间
状态，标明该笔支出是否已经完成； 
投票记录，记录所有投资人在该笔支出上的投票记录，所有投票过的投资人都会被记录下来

```

项目流程
```shell
# 创建项目
参与众筹，参与的含义是投资人选定某个项目，并向智能合约转账，智能合约会把投资人记录在投资人列表中，并更新项目的资金余额
# 创建资金支出条目:项目所有者有权限在项目上发起资金支出请求，需要提供资金用途、支出金额、收款方，
默认为未完成状态，创建资金支出条目前需要检查资金余额是否充足

# 给资金支出条目投票，投资人看到新的资金支出请求之后会选择投赞成票还是反对票，投票过程需要被如实记录，
为了简化，我们只记录赞成票

# 完成资金支出，项目所有者在资金支出请求达到超过半数投资人投赞成票的条件时才有权进行此操作，操作结果是直接把对应的金额转给收款方，转账前也要进行余额检查

```

## 智能合约开发

我们可以利用[remix](https://remix.ethereum.org/)编写我们的智能合约

下面是我们的智能合约代码
```
pragma solidity ^0.4.25;

//保证合约内部的四则运算的安全，常见的即使一个数超出uint256后会变成一个最小的数
library SafeMath { 
    function mul(uint a, uint b) internal pure returns (uint) { 
        uint c = a * b; 
        //如果a是0,那么c肯定是0，不用考虑溢出
        //如果c / a == b 那么肯定也没有溢出，溢出的话c一般是一个很小的数
        assert(a == 0 || c / a == b); 
        return c;
    }
    function div(uint a, uint b) internal pure returns (uint) { 
        //b=0的话solidity本身就会报错
        // assert(b > 0); // Solidity automatically throws when dividing by 0 
        uint c = a / b; 
        //不存在溢出
        // assert(a == b * c + a % b); // There is no case in which this doesn't hold 
        return c; 
    } 
    function sub(uint a, uint b) internal pure returns (uint) { 
        //要求返回的是一个无符号的整数
        assert(b <= a); 
        return a - b; 
    } 
    function add(uint a, uint b) internal pure returns (uint) { 
        uint c = a + b;
        //加法要溢出，c就是一个很小的数，b是一个无符号的整数
        assert(c >= a); 
        return c; 
    } 
}

//这里创建这个合约是让任何一个人都可以利用这个合约创建多个项目，将合约作为一个平台
contract ProjectList{
    //对合约内部变量uint使用安全库
    using SafeMath for uint;
    //一个用户的所有的创建的项目的列表
    address[] public Projects;
    // mapping(uint=>address) public Projects;
    // uint ProjectsCount;
 
  //合约内部调用内部合约
    function createProject(string _description,uint _minInvest, uint _maxInvest,uint _goal) public {
        //同一个用户即使每次传入的参数相同 new Project(..)都会生成一个新的项目地址
        address newProject = new Project(msg.sender,_description,_minInvest,_maxInvest,_goal);
        //为什么不用mapping,mapping是没有办法遍历返回的
        Projects.push(newProject);
        // ProjectsCount+=1;
        // Projects[ProjectsCount] = newProject;
    }
    
    //返回用户所有的项目地址是一个数组
    function getProjects() public view returns(address[]) {
        return Projects;
    }
  
}

contract Project{
    //表示将SafeMath库绑定到一个uint类型上，那么uint类型就可以调用SafeMath库里面的方法
    //谁调用了SafeMath，那么谁是 SafeMath函数里面的第一个参数a
    using SafeMath for uint;
    //支付请求的结构体
    struct Payment{
        string description; //支付描述
        uint amount;        //支付请求金额
        address receiver;   //收款地址
        bool completed;     //支付请求状态
        mapping(address=>bool) voters; //投资人的同意状态，新建Payment类型变量，不指定默认的mapping的value值是0
        uint voterCount;    //已经处理支付请求的投资人数量
    }
    
    address public owner;       //融资新项目地址
    //为了防止输入的描述过大，我们可以将其类型设为定长的，这里暂时不更改
    string public description;  //新项目描述  
    uint public minInvest;      //新项目最小投资额
    uint public maxInvest;      //新项目最大投资额
    uint public goal;           //新项目投资金额上限
    mapping(address=>uint) public investors;    //投资人地址=>投资金额
    uint public investorCount; //投资人的总数
    Payment[] public payments;  //新项目的所有的支付请求列表数组
    
    //权限检查，modifier一般在我们的构造函数前面定义，
    modifier ownerOnly(){
         require(msg.sender == owner,"message send");
         //占位符，表示我们应用ownerOnly的这个函数修饰器的函数的本身的代码就会在占位符下面执行
         //函数修饰器相当于在该函数前面添加了一行代码
         _;
    }

    //注意这个地址是由上级合约传入的
    //指定部署合约初始化时从外面接收到参数值
    constructor(address _owner,string _description,uint _minInvest, uint _maxInvest,uint _goal) public  {    
        owner = _owner;
        description = _description;
        minInvest = _minInvest;
        maxInvest =_maxInvest;
        goal = _goal;
    }
    

    //投资函数
    function contribute() public payable{
        //投资金额大于最低投资金额，小于最大投资金额，目前总的投资总额小于总投资
        require(msg.value >= minInvest,"投资金额小于invest value should be larger than minInvest");
        require(msg.value <= maxInvest,"invest value shoule be less than maxInvest");
        
        //uint newBalance = address(this).balance.add(msg.value);
        //require(newBalance <= goal,"total amount shoule not exceed goal");
        require(address(this).balance <= goal,"total amount shoule not exceed goal");
        if(investors[msg.sender] == 0){
              investorCount++;  
        }
        //更新投资人列表mapping和总投人的数量，可能是一个投资人追加投资
        investors[msg.sender] += msg.value;
        
    }
    
    //新项目中创建支付请求
    //ownerOnly表示上面的权限检查，只有项目方才能创建资金支出请求
    function createPayment(string _description,uint _amount,address _receiver) public ownerOnly{
        //因为这里只是创建一个临时数据，只用memory就可以，默认是storeage类型
        Payment memory newPayment = Payment({
            description:_description,
            amount:_amount,
            receiver:_receiver,
            completed:false,
            voterCount:0
        });
        payments.push(newPayment);
    } 
    
    //投资人针对哪个支出请求投票
    function voteForPayment(uint index) public {
        //这里需要对已经存在的数据进行更新，需要用storage，说明是一个引用的更改
        Payment storage payment = payments[index];
        //判断投票者 是不是投资人
        require(investors[msg.sender] > 0,"msg sender should be investor.");
        //判断该投资人是否已经投过票
        require(!payment.voters[msg.sender],"message sender shoule not has voted");
        //更新投票状态
        payment.voters[msg.sender] = true;
        //更新投票的总数
        payment.voterCount += 1;
    }

    //对某个支出请求确认支付
    //ownerOnly表示上面的权限检查，只有项目方才能执行转账操作
    function doPayment(uint index) public ownerOnly{
        
        Payment storage payment = payments[index];
        //判断该笔请求还没有被支付
        require(!payment.completed,"payment shoule not be completed");
        //至少需要一半以上的投资人同意该笔支付请求
        require(payment.voterCount > (investorCount/2),"voters shoue be more than 1/2 of investors.");
        //合约余额大于请求付款金额
        require(address(this).balance >= payment.amount,"balance shoule be larger than payment");
        //付款给receiver
        payment.receiver.transfer(payment.amount);
        //更新支付请求的的付款状态
        payment.completed = true;
        
    }
    
}

``` 
注意的几点：
```
合约一般不要有循环，可能会引起gas过大
注意检查余额，注意安全机制
注意几个可能消费gas过大的地方，可以适当修改
合理的利用mapping类型解决很多数据问题
```

## 编写合约遇到的坑

余额添加

## 编译脚本

首先安装编译工具和fs-extra包(主要用来处理文件读取) 
```
npm install solc 
npm install fs-extra
```
>如果出现编译错误，在检查合约无误之后，可能是solc的版本有问题，改成npm install solc@0.4.25

在scripts下新建compile.js脚本

```js
const fs = require('fs-extra');
const solc = require('solc');
//node的默认模块
const path = require('path');
//在脚本执行的开始加入清除编译结果的代码，__dirname是当前的目录
const compiledPath = path.resolve(__dirname, '../compiled');
fs.removeSync(compiledPath); 
//确保compiledPath目录存在，如果不存在就新建
fs.ensureDirSync(compiledPath);
//
const contractFiles = fs.readdirSync(path.resolve(__dirname, '../contracts'));
contractFiles.forEach(contractFile => {
	//编译
    //拼接一个完整的路径
	const contractPath = path.resolve(__dirname, '../contracts', contractFile);
    //同步方式读取文件
	const contractSource = fs.readFileSync(contractPath, 'utf-8');
    //后面的1表示打开solc里面的优化器
	let compileResult = solc.compile(contractSource, 1);

    //判断错误，如果源代码有问题，我们在编译阶段就应该报出来，而不应该把错误的结果写入到文件系统，因为这样会导致后续步骤失败
	if (Array.isArray(compileResult.errors) && compileResult.errors.length) {
		throw new Error(compileResult.errors[0]);
	}

	Object.keys(compileResult.contracts).forEach(name => {
        //以冒号开头的全部替换成空
		let contractName = name.replace(/^:/, '');
        //定义一个路径，注意反引号使用，${contractName}表示反引号中引用的一个变量
		let filePath = path.resolve(compiledPath, `${contractName}.json`);
        //把编译后的合约的结果以json方式存入filePath
		fs.outputJsonSync(filePath, compileResult.contracts[name]);
		console.log("Saving json file to ", filePath);
	});
});

```

修改package.json
```
  "scripts": {
   "compile": "node scripts/compile.js"
},
```
这样我们就可以使用`npm run compile`来进行编译,
```shell
cd vote_DApp
npm run compile
```
如果正常的话你将看到如下结果
```shell
Saving json file to  /root/ethProject/project/project_DApp/compiled/Project.json
Saving json file to  /root/ethProject/project/project_DApp/compiled/ProjectList.json
Saving json file to  /root/ethProject/project/project_DApp/compiled/SafeMath.json
```

## 部署脚本

安装web3
```shell
npm install web3
```
web3中有eth对象，通过`web3.eth`对象可以与以太坊区块链进行交互。

我们首先需要编辑一个配置文件，我们可以随时更新我们的配置文件，方便其他文件读取
```shell
cd vote_DApp
mkdir config
vim default.js
```
内容如下
```js
module.exports = {
    providerUrl: 'http://localhost:8545'
};
```
然后编辑我们的
```js
const Web3 = require('web3')
// const web3 = new Web3(new Web3.providers.HttpProvider("http://localhost:8545"));
const config = require('config');
const web3 = new Web3(new Web3.providers.HttpProvider(config.get('providerUrl')));
const fs = require('fs-extra');
const path = require('path');
// 1. get bytecode
const filePath = path.resolve(__dirname, '../compiled/ProjectList.json');
//ES6的匹配机制
const {interface, bytecode} = require(filePath);

(async () =>{
    // 2. get accounts
	let accounts = await web3.eth.getAccounts();
	// 3. get contract instance and deploy
	console.time("deploy time");
    //deploy({data:合约的字节码,agruments:['AUDI']})
	let result = await new web3.eth.Contract(JSON.parse(interface))
				 .deploy({data:'0x' + bytecode})
				 .send({from: accounts[0], gas: 5000000});
	console.timeEnd("deploy time");

    //获取编译生成的合约地址
	const contractAddress = result.options.address;
	console.log("contract address: ", contractAddress);
    //拼接路径
	const addressFile = path.resolve(__dirname, '../address.json');
    //将地址写入文件
	fs.writeFileSync(addressFile, JSON.stringify(contractAddress));
	console.log("write address successfully to: ", addressFile);
	process.exit();
})();
```

修改`package.json`,注意标点符号
```js
 "scripts": {
   "compile": "node scripts/compile.js",
   "predeploy": "npm run compile",
   "deploy": "node scripts/deploy.js"
}
```
执行
```shell
cd vote_DApp
npm run deploy
```
正常的化可以看到如下结果
```
deploy time: 63.055ms
contract address:  0x625E0E1F224f8414208267fc533edae1Cb758F35
write address successfully to:  /root/ethProject/project/project_DApp/address.json

> project_dapp@1.0.0 deploy /root/ethProject/project/project_DApp
> node scripts/deploy.js

deploy time: 82.602ms
contract address:  0x98Eeee8C3D70145b56F5dF332Ee4E8cC57cF713c
write address successfully to:  /root/ethProject/project/project_DApp/address.json
```
`address.json`就保存了我们创建的合约的地址



## 安装Next.js和React
React，一个构建用户界面的JavaScript库，主要用于构建UI，负责视图层和简单的状态管理；

Next，一个轻量级的 React 服务端渲染应用框架。负责后端请求的处理，把 React 生态里面的各种工具帮开发者拼起来，极大的降低了 React SSR 的上手门槛，同时使用 next-routes 来实现用户友好的 URL；

Material UI，负责提供开箱即用的样式组件，相比 Semantic UI、Elemental、Element React 等更新更活跃，接触和熟悉的开发者群体更大
```
npm install --save next react react-dom
```
修改package.json，添加用于启动项目和构建项目的npm script
```js
"scripts": {
	...
    "dev": "next",
    "build": "next build",
    "start": "next start"
  },

```
## 创建项目首页
```
cd vote_DApp
添加pages/index.js
```
编辑`index.js`
```js
//ES6引入包
import React from 'react';
//export default将后面的内容暴露出去，被其他引用
//React和 Vue中很多功能都是通过一个一个组件组合构造合成
//可以认为这里在编写一个继承了React.Component的组件
//render()函数是组件index中的主函数，主要做页面的渲染，这里暴露出去一个html页面
export default class Index extends React.Component { 
    render() { 
    //这是jsx语法，{ }里面的内容作为js解析， <>里面的内容会作为html解析
    return <div>Welcome to Ethereum ICO DApp!</div>;
}
```
修改`package.json`
```
"scripts": {
    ...
    "dev": "next",
    "build":"next build",
    "start": "next start"
 },

```
测试 next
```
cd vote_DApp
npm run dev
```
正常的话你应该看到
```shell
Ready on http://localhost:3000
```
如果端口被占用，你也可以通过`npm run dev -- -p 4000`指定其他的端口

## 设计路由
在next.js 默认路由机制中，项目详情页的路由会是这样：`/projects?address=0x123456`,我们可以将它改进为如下路由：
`/projects/0x123456`
使用 next-routes 能够很方便的实现用户友好的路由，步骤如下

1. 安装依赖
```
npm install --save next-routes
```
2.根目录下增加routes.js
```js
//返回的是一个函数，所以需要调用，拿到函数调用的结果
const routes = require('next-routes')();
routes 
 //根据第一个参数访问的路由，后面跳转页面
    .add('/projects/create', 'projects/create')
    .add('/projects/:address', 'projects/detail')
    .add('/projects/:address/payments/create', 'projects/payments/create');
module.exports = routes
```
3，根目录增加server.js
```js
//引入依赖
const next = require('next');
const http = require('http');
const routes = require('./routes');
//我们知道next自己会启动一个server,我们怎么将server和路由连接起来？通过
//routes.getRequestHandler将服务器和路由进行绑定
const app = next({ dev: process.env.NODE_ENV !== 'production' });
const handler = routes.getRequestHandler(app);
//启动服务器
app.prepare().then(() => {
    http.createServer(handler).listen(3000, () => {
        console.log('server started on port: 3000');
    });
});

```
修改package.json我们现在不是在启动netx服务器了，next服务已经被封装在server.js这个文件，而是启动server.js这个服务了。
```js
"scripts": {
    "compile": "node scripts/compile.js",
    "predeploy": "node scripts/deploy.js",
    "deploy": "node scripts/deploy.js",
    "dev": "node server.js",
    "build": "next build",
    "start": "NODE_ENV=production node server.js"
  }
```
` "start": "NODE_ENV=production node server.js"`这一项,就是传入一个环境变量
和`const app = next({ dev: process.env.NODE_ENV !== 'production' })`形成对应,

dev是false表示启动生产环境，是true表示启动调试模式环境

## 测试
```js
npm run dev
```
正常的化，你将看到如下结果
```shell
Compiled successfully!

Note that pages will be compiled when you first load them.
server started on port:8080

```
## 集成Material UI

## .安装依赖

```
npm install --save @material-ui/core@1.5.1
```
material-ui/core这个库更新比较快，这里我们采用1.5.1版本，不同的版本可能用法也不太一样

## 导入支持服务端渲染的必要文件

官方示例项目中有两个支持服务端渲染的必要文件：withRoot.js、getPageContext.js 需要复制到我们的项目中。

在项目跟目录下新建 libs 目录，然后将这两个文件原封不动复制过来即可，两个文件的作用描述如下
```js
cd vote_DApp
mkdir libs
vim withRoot.js        # 粘贴内容
vim getPageContext.js  # 粘贴内容
```
withRoot.js 实质上是 React 组件，并且是高阶组件，作用在于确保服务端渲染时能正确的生成样式

`withRoot.js`
```js
//引入依赖
import React from 'react';
import PropTypes from 'prop-types';
import { MuiThemeProvider } from '@material-ui/core/styles';
import CssBaseline from '@material-ui/core/CssBaseline';
import getPageContext from './getPageContext';

function withRoot(Component) {
    //定义一个继承了React.Component组件的高阶组件类
  class WithRoot extends React.Component {
    //构造函数
    constructor(props, context) {
      super(props, context);

      this.pageContext = this.props.pageContext || getPageContext();
    }

    componentDidMount() {
      // Remove the server-side injected CSS.
      const jssStyles = document.querySelector('#jss-server-side');
      if (jssStyles && jssStyles.parentNode) {
        jssStyles.parentNode.removeChild(jssStyles);
      }
    }

    pageContext = null;


    render() {
      // MuiThemeProvider makes the theme available down the React tree thanks to React context.
      //theme={this.pageContext.theme}知道我们引入的主题
      return (
        <MuiThemeProvider theme={this.pageContext.theme} sheetsManager={this.pageContext.sheetsManager}>
          {/* CssBaseline kickstart an elegant, consistent, and simple baseline to build upon. */}
          <CssBaseline />
          {/* 渲染传入的组件 */}
          <Component {...this.props} />
        </MuiThemeProvider>
      );
    }
  }

  WithRoot.propTypes = {
    pageContext: PropTypes.object,
  };

  WithRoot.getInitialProps = ctx => {
    if (Component.getInitialProps) {
      return Component.getInitialProps(ctx);
    }

    return {};
  };

  return WithRoot;
}
//向外暴露函数withRoot
export default withRoot;

```

getPageContext.js 为整个应用的前端、后端渲染提供 theme 配置，以及必要的工具函数，比如如何生成类名等

## 创建自定义的页面结构

参照示例项目中的` pages/_document.js`，是基于` Next.js `里面的 `Custom Document` 机制实现的，创建自定义页面结构的动机在于我们可以全局性的在页面中引入需要的样式和字体文件,简单来说我们可以通能过`_document.js`来自定义我们页面的一些信息，

具体参见 `pages/_document.js`

##  使用Material UI中的组件
接下来检验 Material UI 的集成是否成功，改动pages/index.js中的内容，其中调用了组件库中的 Button 组件：
```js
import React from 'react';
import { Button } from '@material-ui/core';
import withRoot from '../libs/withRoot';

class Index extends React.Component {
render() {
    //primary是之前我们在getPageContext.js 文件中定义的样式    
    return <Button variant="raised" color="primary">Welcome to Ethereum vote_DApp!</Button>;
    }
}
export default withRoot(Index);
```

## 重启服务

执行 `npm run dev`，刷新页面，应该看到添加了Button组件的首页.成功的话你将看到如下信息
![]("")

## layout组件
layout组件主要就是规范我们页面的宽度,高度等
```
cd vote_DApp
mkdir components
vim Header.js
```
可以去这个地址看到相关内容

修改pages/index.js 应用Layout
```js
import React from 'react';
import { Button } from '@material-ui/core';
import withRoot from '../libs/withRoot';
import Layout from '../components/Layout';

class Index extends React.Component {
    render() {
        return (
            <Layout>
                <Button variant="raised" color="primary">
                    Welcome to Ethereum ICO DApp!
                </Button>
            </Layout>
        )
    }
}
export default withRoot(Index);
```



## Header组件
主要定义我们页面公用的页面顶部样式
```
cd vote_DApp
mkdir components
vim Header.js
```
因为 Header 组件也是全局通用的，在 Layout 中引入即可,可以去这个地址看到相关内容 

```js
import React from 'react';
import { withStyles } from '@material-ui/core/styles';
import Header from './Header';
...
class Layout extends React.Component {
render() {
const { classes } = this.props;
return (
        <div className={classes.container}>
        <Header />
        <div className={classes.wrapper}>{this.props.children}</div>
        </div>
    );
}

```
到现在整体目录如下
```shell
|── components
|   |── Header.js 
|   └── Layout.js 
|── libs
|   |── getPageContext.js
|   └── withRoot.js
|── package.json
|── pages
|   |── _document.js
|   └── index.js 
|── routes.js
|── server.js

```


## 区块链交互

## 创建前后端通用的web3实例

为了解决服务端渲染带来的Web3对象缺失问题，我们需要兼容前后端的去创建 Web3 实例，产生在浏览器环境和在 Node.js 环境都可以使用 Web3 实例，直接在 libs 目录下新建 web3.js，
```shell
cd libs
vim web3.js
```
内容如下
```shell
import Web3 from 'web3';
let web3;

// if browser enviroment & Metamask exists
if (typeof window !== 'undefined' && typeof window.web3 !== 'undefined') {
web3 = new Web3(window.web3.currentProvider);
} else {
web3 = new Web3(new Web3.providers.HttpProvider('http://localhost:8545'));
}
```
在初始化 Web3 示例时会检查当前的环境，如果是浏览器环境并且用户安装了Metamask，直接使用Metamask注入的Provider，否则使用HttpProvider通过本地的以太坊节点与区块链网络进行通信。

## 封装合约实例

在 libs 目录下新建 projectList.js，
```js
import web3 from './web3';
import ProjectList from '../compiled/ProjectList.json';
import address from '../address.json';

const contract = new web3.eth.Contract(JSON.parse(ProjectList.interface), address);

export default contract;
```
这是部署合约后获取projectlist的合约实例
## 封装项目合约实例

一个projectlist的合约实例下面可能有很多的项目。

在 libs 目录下新建 project.js文件
```js
import web3 from './web3';
import Project from '../compiled/Project.json';

const getContract = address => 
		new web3.eth.Contract(JSON.parse(Project.interface), address);

export default getContract;

```

后续步骤就比较容易了，主要是react语法，js获取从区块链上获取数据然后填充数据，可以参照这个地址获取后续内容
