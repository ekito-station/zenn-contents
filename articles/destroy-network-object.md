---
title: "他人の生成したネットワークオブジェクトを削除する（PUN2）"
emoji: "🗑️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["unity", "csharp", "photon", "pun2", "個人開発"]
published: true
---
# はじめに
PUN2（Photon Unity Networking 2）において、ネットワークオブジェクトを削除するには通常`PhotonNetwork.Destroy`を使用します。
ただ、他のプレイヤーが生成したネットワークオブジェクトなど自分が所有権を持たないネットワークオブジェクトは、`PhotonNetwork.Destroy`だけでは削除できません。所有権を自分に移譲してから`PhotonNetwork.Destroy`を使用する必要があります。そこで、以下に載せる方法を試しました。

# 1. ネットワークオブジェクトの所有権オプションの変更
ネットワークオブジェクトにアタッチされている`PhotonView`コンポーネントにおいて、`Ownership Transfer`のオプションを変更します。デフォルトでは`Fixed`になっていますが、`Takeover`に変更します。
`Fixed`では、ネットワークオブジェクトの所有権は生成者が常に持ち他のプレイヤーは取得できませんが、`Takeover`では他のプレイヤーも所有権を取得できるようになります。
![](/images/destroy-network-object/image1.png)

# 2. TransferOwnershipを呼び出す
ネットワークオブジェクトを削除したい時に呼び出す関数の中で、`TransferOwnership`を呼び出します。これによって、所有権を移譲することができます。

```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Photon.Pun;
using Photon.Realtime;

public class DestroyNetworkObject : MonoBehaviourPunCallbacks
{
    private void DestroyObj(GameObject obj)
    {
        // 引数として渡されたゲームオブジェクトのPhotonViewを取得し
        // その所有権を自分（PhotonNetwork.LocalPlayer）に移譲する
        obj.GetComponent<PhotonView>().TransferOwnership(PhotonNetwork.LocalPlayer);
    }
}
```

# 3. 所有権移譲後にネットワークオブジェクトを削除する
所有権の移譲が行われた時に呼ばれるコールバックの中で、ネットワークオブジェクトの削除を行います。
まず、スクリプトに`IPunOwnershipCallbacks`インターフェースを実装することで、所有権関連のコールバックを受け取ることができます。
次に、`IPunOwnershipCallbacks.OnOwnershipTransfered`の中に、ネットワークオブジェクトを削除する処理を追加します。
```csharp
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using Photon.Pun;
using Photon.Realtime;

// IPunOwnershipCallbacksを追加
public class DestroyNetworkObject : MonoBehaviourPunCallbacks, IPunOwnershipCallbacks
{
    private void DestroyObj(GameObject obj)
    {
        obj.GetComponent<PhotonView>().TransferOwnership(PhotonNetwork.LocalPlayer);
    }
    // IPunOwnershipCallbacks.OnOwnershipTransferedを実装
    void IPunOwnershipCallbacks.OnOwnershipTransfered(PhotonView targetView, Player previousOwner) 
    {
        // ネットワークオブジェクトを削除
        PhotonNetwork.Destroy(targetView.gameObject);
    }

    // 以下のメソッドも実装しないとエラーが出る
    void IPunOwnershipCallbacks.OnOwnershipTransferFailed(PhotonView targetView, Player previousOwner)
    {

    }

    void IPunOwnershipCallbacks.OnOwnershipRequest(PhotonView targetView, Player requestingPlayer) 
    {

    }
}
```
# おわりに
以上の方法で、他のプレイヤーが生成したネットワークオブジェクトを削除することができました。
[GitHub](https://github.com/ekito-station/destroy-network-object)に、サンプルプロジェクトをアップしました。ランダムな位置に生成されたネットワークオブジェクトをクリックすると削除することができます。
https://github.com/ekito-station/destroy-network-object