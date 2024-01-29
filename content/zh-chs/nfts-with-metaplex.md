# 使用Metaplex创建Solana NFTs

## 简介

Solana的非同质化代币（NFTs）是使用Token程序创建的SPL代币。然而，这些代币还有一个额外的与每个代币铸造相关联的元数据帐户。这允许了广泛的用例，你可以有效地将任何东西令牌化，从游戏库存到艺术品。

在这篇文章中，我们将介绍Solana上NFTs的基本概念，如何使用Metaplex SDK创建和更新它们，并简要介绍一些工具，这些工具可以帮助你在Solana上大规模创建和分发NFTs。

## Solana上的NFTs

Solana上的NFT是一种非可分割代币，并附带有元数据。此外，代币的铸造有一个最大供应量为1。

换句话说，NFT是来自Token程序的标准代币，但与你可能认为的“标准代币”不同，它具有以下特点：

1. 小数位为0，因此不能分割为部分
2. 来自一个最大供应量为1的代币铸造，因此只有1个这样的代币存在
3. 来自一个授权设置为 `null` 的代币铸造（以确保供应量永远不会改变）
4. 有一个存储元数据的关联帐户

虽然前三个特点是使用SPL Token程序可以实现的功能，但关联的元数据需要一些额外的功能。

通常，NFT的元数据既有链上部分又有链下部分。见下图：

![元数据截图](../assets/solana-nft-metaplex-metadata.png)

- **链上元数据** 存储在与代币铸造关联的帐户中。链上元数据包含一个指向链下`.json`文件的URI字段。
- **链下元数据** 在JSON文件中存储NFT的媒体链接（图像、视频、3D文件）、NFT可能具有的特征以及其他元数据。通常使用永久数据存储系统（如Arweave）来存储NFT元数据的链下部分。

## Metaplex

[Metaplex](https://www.metaplex.com/)是一个提供一套工具的组织，比如[Metaplex SDK](https://docs.metaplex.com/sdks/js/)，它简化了在Solana区块链上创建和分发NFTs的过程。这些工具适用于各种用例，并且可以轻松地管理创建和铸造NFT集合的整个过程。

更具体地说，Metaplex SDK旨在帮助开发人员利用Metaplex提供的链上工具。它提供了用户友好的API，专注于流行的用例，并允许与第三方插件轻松集成。要了解Metaplex SDK的功能，请参阅[README](https://github.com/metaplex-foundation/js#readme)。

Metaplex提供的一个重要程序是Token Metadata程序。Token Metadata程序标准化了将元数据附加到SPL代币的过程。使用Metaplex创建NFT时，Token Metadata程序使用代币铸造作为种子，使用程序派生地址（PDA）创建元数据帐户。这使得任何NFT的元数据帐户都可以通过代币铸造的地址确定性地定位。要了解有关Token Metadata程序的更多信息，请参阅Metaplex [文档](https://docs.metaplex.com/programs/token-metadata/)。

在接下来的部分，我们将介绍使用Metaplex SDK准备资产、创建NFT、更新NFT以及将NFT与更大的集合关联的基础知识。

### Metaplex实例

`Metaplex`实例作为访问Metaplex SDK API的入口点。此实例接受用于与集群通信的连接。此外，开发人员可以通过指定“Identity Driver”和“Storage Driver”来定制SDK的交互。

Identity Driver实际上是一个可用于签署交易的密钥对，在创建NFT时是必需的。Storage Driver用于指定要用于上传资产的存储服务。 `bundlrStorage`驱动程序是默认选项，它将资产上传到Arweave，这是一个永久和去中心化的存储服务。

以下是如何为devnet设置`Metaplex`实例的示例。

```tsx
import {
  Metaplex,
  keypairIdentity,
  bundlrStorage,
} from "@metaplex-foundation/js";
import { Connection, clusterApiUrl, Keypair } from "@solana/web3.js";

const connection = new Connection(clusterApiUrl("devnet"));
const wallet = Keypair.generate();

const metaplex = Metaplex.make(connection)
  .use(keypairIdentity(wallet))
  .use(
    bundlrStorage({
      address: "https://devnet.bundlr.network",
      providerUrl: "https://api.devnet.solana.com",
      timeout: 60000,
    }),
  );
```

### 上传资产

在创建NFT之前，您需要准备并上传您计划与NFT关联的任何资产。虽然这不一定是一个图像，但大多数NFT都与图像相关联。

准备和上传图像涉及将图像转换为缓冲区，使用`toMetaplexFile`函数将其转换为Metaplex格式，然后将其上传到指定的Storage Driver。

Metaplex SDK支持从本地计算机上的文件或通过浏览器上传的文件创建新的Metaplex文件。您可以通过使用`fs.readFileSync`来读取图像文件，然后使用`toMetaplexFile`将其转换为Metaplex文件。最后

，使用您的`Metaplex`实例调用`storage().upload(file)`来上传文件。该函数的返回值将是存储图像的URI。

```tsx
const buffer = fs.readFileSync("/path/to/image.png");
const file = toMetaplexFile(buffer, "image.png");

const imageUri = await metaplex.storage().upload(file);
```

### 上传元数据

在上传图像之后，是时候使用`nfts().uploadMetadata`函数上传链下JSON元数据了。这将返回存储JSON元数据的URI。

请记住，元数据的链下部分包括诸如图像URI以及NFT的名称和描述等附加信息。虽然你可以在这个JSON对象中包含任何你想要的东西，但在大多数情况下，你应该遵循[NFT标准](https://docs.metaplex.com/programs/token-metadata/token-standard#the-non-fungible-standard)以确保与钱包、程序和应用程序的兼容性。

要创建元数据，请使用SDK提供的`uploadMetadata`方法。此方法接受一个元数据对象，并返回一个指向上传的元数据的URI。

```tsx
const { uri } = await metaplex.nfts().uploadMetadata({
  name: "My NFT",
  description: "My description",
  image: imageUri,
});
```

### 创建NFT

上传NFT的元数据后，您可以在网络上创建NFT。Metaplex SDK的`create`方法允许您使用最小的配置创建一个新的NFT。该方法将处理为您创建的代币铸造帐户、代币帐户、元数据帐户和主版本帐户。提供给此方法的数据将表示NFT元数据的链上部分。您可以探索SDK以查看此方法可以可选提供的所有其他输入。

```tsx
const { nft } = await metaplex.nfts().create(
  {
    uri: uri,
    name: "My NFT",
    sellerFeeBasisPoints: 0,
  },
  { commitment: "finalized" },
);
```

此方法返回一个包含有关新创建的NFT的信息的对象。默认情况下，SDK将`isMutable`属性设置为true，允许对NFT的元数据进行更新。但是，您可以选择将`isMutable`设置为false，使NFT的元数据不可变。

### 更新NFT

如果您将`isMutable`设置为true，您可能有理由更新NFT的元数据。SDK的`update`方法允许您更新NFT的链上和链下元数据。要更新链下元数据，您需要重复之前提到的上传新图像和元数据URI的步骤，然后将新的元数据URI提供给此方法。这将更改链上元数据指向的URI，从而有效地更新链下元数据。

```tsx
const nft = await metaplex.nfts().findByMint({ mintAddress });

const { response } = await metaplex.nfts().update(
  {
    nftOrSft: nft,
    name: "Updated Name",
    uri: uri,
    sellerFeeBasisPoints: 100,
  },
  { commitment: "finalized" },
);
```

请注意，您在调用`update`时不包括的任何字段将保持不变。

### 将NFT添加到集合

[Certified Collection](https://docs.metaplex.com/programs/token-metadata/certified-collections#introduction)是一个可以包含单个NFT的NFT。想象一个大型的NFT集合，比如Solana Monkey Business。如果您查看单个NFT的[Metadata](https://explorer.solana.com/address/C18YQWbfwjpCMeCm2MPGTgfcxGeEDPvNaGpVjwYv33q1/metadata)，您将看到一个`collection`字段，其中的`key`指向`Certified Collection` [NFT](https://explorer.solana.com/address/SMBH3wF6baUj6JWtzYvqcKuj2XCKWDqQxzspY12xPND/)。简而言之，作为集合的一部分的NFT与另一个代表集合本身的NFT相关联。

要将NFT添加到集合中，首先必须创建Collection NFT。此过程与之前相同，除了在我们的NFT元数据上包括一个附加字段：`isCollection`。此字段告诉token程序，此NFT是Collection NFT。

```tsx
const { collectionNft } = await metaplex.nfts().create(
  {
    uri: uri,
    name: "My NFT Collection",
    sellerFeeBasisPoints: 0,
    isCollection: true
  },
  { commitment: "finalized" },
);
```

然后，您将集合的铸造地址列为我们新NFT中的`collection`字段的参考。

```tsx
const { nft } = await metaplex.nfts().create(
  {
    uri: uri,
    name: "My NFT",
    sellerFeeBasisPoints: 0,
    collection: collectionNft.mintAddress
  },
  { commitment: "finalized" },
);
```

当您查看新创建的NFT的元数据时，您应该看到一个类似如下的`collection`字段：

```JSON
"collection":{
  "verified": false,
  "key": "SMBH3wF6baUj6JWtzYvqcKuj2XCKWDqQxzspY12xPND"
}
```

您需要做的最后一件事是验证NFT。这实际上只是将上述`verified`字段翻转为true，但这非常重要。这是让消费程序和应用程序知道您的NFT实际上是集合的一部分的关键。您可以使用`verifyCollection`函数来执行此操作：

```tsx
await metaplex.nfts().verifyCollection({
  mintAddress: nft.address,
  collectionMintAddress: collectionNft.address,
  isSizedCollection: true,
})
```

### 糖果机（Candy Machine）

当创建和分发大量的NFT时，Metaplex通过他们的[Candy Machine](https://docs.metaplex.com/programs/candy-machine/overview)程序和[Sugar CLI](https://docs.metaplex.com/developer-tools/sugar/overview)使这变得简单。

Candy Machine实际上是一个铸造和分发程序，帮助启动NFT收藏品。Sugar是一个命令行界面，帮助您创建糖果机，准备资产，并批量创建NFT。上面介绍的创建NFT的步骤，如果一次创建数千个NFT将会非常繁琐。Candy Machine和Sugar通过提供多项保障来解决这个问题，并确保公平启动。

我们不会深入介绍这些工具，但绝对可以查看[Metaplex文档中关于Candy Machine和Sugar如何协同工作的内容](https://docs.metaplex.com/developer-tools/sugar/overview/introduction)。

要探索Metaplex提供的全部工具，请访问GitHub上的[Metaplex存储库](https://github.com/metaplex-foundation/metaplex)。