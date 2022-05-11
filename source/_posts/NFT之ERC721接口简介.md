---
title: NFT之ERC721接口简介
date: 2022-05-11 10:50:37
tags:
  - 区块链
  - NFT
  - ERC721
  - ERC20
categories:
  - 区块链
---

NFT是*non-fungible tokens*的缩写，与之对应的是可互换代币*fungible tokens*

NFT标准**草案**ERC721是可互换代币标准ERC20的扩展，了解ERC20有利于了解ERC721。

本文主要内容

- ERC20接口简介
- ERC721接口简介

<!--more-->

## ERC20

```
function name() public view returns (string)
function symbol() public view returns (string)
function decimals() public view returns (uint8)
function totalSupply() public view returns (uint256)
function balanceOf(address _owner) public view returns (uint256 balance)
function transfer(address _to, uint256 _value) public returns (bool success)
function transferFrom(address _from, address _to, uint256 _value) public returns (bool success)
function approve(address _spender, uint256 _value) public returns (bool success)
function allowance(address _owner, address _spender) public view returns (uint256 remaining)

```

- name()
	代币名称，比如“人民币”
- symbol()
	代币符号，比如RMB
- decimals()
	代币小数位，代币都是以uint256表示的，小数位只影响显示
- totalSupply()
	代币的发行总量
- balanceOf()
	某人（某地址）拥有的代币数量
- transfer()
	代币拥有者向某人（某地址）转若干代币
- transferFrom
	代币拥有者的代理人，想某人（某地址）转若干代币。注意这里的**代理人**
- approve()
	代币拥有者授权某人（某地址）为代理人，同时指定代理额度。
- allowance()
	查询某代理人的剩余代理额度
	

所谓**代理人**，是代替代币拥有者进行操作的人，在ERC721也延续了这类接口。

## ERC721

开始了解ERC721的时候，容易有两个疑问：

1. approve是什么意思
2. 为什么没有用来mint的接口

下面来试着解答这两个问题

Methods
```
    function balanceOf(address _owner) external view returns (uint256);
    function ownerOf(uint256 _tokenId) external view returns (address);
    function safeTransferFrom(address _from, address _to, uint256 _tokenId, bytes data) external payable;
    function safeTransferFrom(address _from, address _to, uint256 _tokenId) external payable;
    function transferFrom(address _from, address _to, uint256 _tokenId) external payable;
    function approve(address _approved, uint256 _tokenId) external payable;
    function setApprovalForAll(address _operator, bool _approved) external;
    function getApproved(uint256 _tokenId) external view returns (address);
    function isApprovedForAll(address _owner, address _operator) external view returns (bool);

```

Events
```
    event Transfer(address indexed _from, address indexed _to, uint256 indexed _tokenId);
    event Approval(address indexed _owner, address indexed _approved, uint256 indexed _tokenId);
    event ApprovalForAll(address indexed _owner, address indexed _operator, bool _approved);
```

了解ERC20后，也就了解了ERC721中很多接口。如`balanceOf`、`Transfer	、`trasferFrom`、`approve`等。

- balanceOf()
	获取某人（某地址）拥有的所有NFT
- ownerOf()
	查看某NFT的拥有者
- transfer()
	转移NFT
- safeTransferFrom()
	安全转移，会检查to、tokenId等参数的有效性
- approve()
	NFT拥有者将某NFT授权给某代理人
- getApproved()
	查询某NFT的代理人
- setApprovalForAll
	撤销/指定某代理人的权限
- approveForAll()
	NFT拥有者将自己所有NFT授权给某代理人

### 答案
现在，**第1个问题**很容易回答了，approve其实是在做授权操作，授权给代理人进行操作。

至于**第2个问题**，ERC721本身没有加入mint操作，需要在编写合约的时候按需加入。

比如cryptopunks项目，其“mint”函数是：
```
function getPunk(uint punkIndex) {
        if (!allPunksAssigned) throw;
        if (punksRemainingToAssign == 0) throw;
        if (punkIndexToAddress[punkIndex] != 0x0) throw;
        if (punkIndex >= 10000) throw;
        punkIndexToAddress[punkIndex] = msg.sender;
        balanceOf[msg.sender]++;
        punksRemainingToAssign--;
        Assign(msg.sender, punkIndex);
    }
```

也就是通过调用该函数直接免费认领NFT

笔者写文章时，下图NFT价值两千三百多万美元，
早知今日，悔不当初，猛狗落泪……
![](https://cryptopunks.app/public/images/cryptopunks/punk5822.png)

在比如房产NFT *otherside*的mint函数,采用了白名单的方式

```
function mintLands(uint256 numLands, bytes32[] calldata merkleProof) external whenPublicSaleActive nonReentrant {
        require(numLands > 0, "Must mint at least one beta");
        require(currentNumLandsMintedPublicSale + numLands <= MAX_PUBLIC_SALE_AMOUNT, "Minting would exceed max supply");
        require(numLands <= maxMintPerTx, "numLands should not exceed maxMintPerTx");
        require(numLands + mintedPerAddress[msg.sender] <= maxMintPerAddress, "sender address cannot mint more than maxMintPerAddress lands");
        if(isKycCheckRequired) {
            require(MerkleProof.verify(merkleProof, kycMerkleRoot, keccak256(abi.encodePacked(msg.sender))), "Sender address is not in KYC allowlist");
        } else {
            require(msg.sender == tx.origin, "Minting from smart contracts is disallowed");
        }
     
        uint256 mintPrice = getMintPrice();
        IERC20(tokenContract).safeTransferFrom(msg.sender, address(this), mintPrice * numLands);
        currentNumLandsMintedPublicSale += numLands;
        mintedPerAddress[msg.sender] += numLands;
        emit PublicSaleMint(msg.sender, numLands, mintPrice);
        mintLandsCommon(numLands, msg.sender);
    }
```

当然也可以采用和上述不同的方案，看合约需求而定。
