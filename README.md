## Linuxサーバーの運用監視と不要サービス・ログの最適化

自宅サーバー（Ubuntu）の構築・運用プロセスにおいて、実務を想定した障害検知の訓練および、システムログのクリーンアップ（ノイズ削減）を実施しました。

### 1. 課題 (Issue)
起動時のシステムジャーナルを調査したところ（`journalctl -p err -b`）、以下の不要なエラーログが継続的に出力されていることを検知しました。
* NFS（Network File System）関連デーモンの起動失敗エラー
* Bluetoothプロファイル（SAP）の初期化失敗エラー

### 2. 調査と原因特定 (Investigation)
* **NFS関連:** 自宅環境ではファイル共有にSamba（SMB）を採用しており、NFS機能自体を一切使用していません。しかし、OSのデフォルト設定でNFSサーバー関連サービスが有効になっており、初期化時に設定ファイルやパイプファイルが見つからずにエラーを出力していました。
* **Bluetooth関連:** サーバー用途のOSであるためBluetooth機能を使用していないにもかかわらず、特定の古いプロファイル（SAP）が初期化を試みて権限エラーを起こしていました。

### 3. 対応とチューニング (Resolution)
不要なサービスを特定し、システムのリソース節約および起動時エラーログの解消（ノイズの排除）を目的に、サービスの停止と自動起動の無効化を実施しました。

**NFSサービスの無効化手順**
```
# 現在動いているNFSサーバーのサービスを直ちに停止する
sudo systemctl stop nfs-server

# 次回のOS起動時にNFS関連サービスが自動で立ち上がらないように無効化する
sudo systemctl disable nfs-server nfs-client.target
```
Bluetoothサービスの無効化手順
```
# 現在動いているBluetoothサービスを直ちに停止する
sudo systemctl stop bluetooth.service

# 次回のOS起動時にBluetoothサービスが自動で立ち上がらないように無効化する
sudo systemctl disable bluetooth.service
```
4. 検証と成果 (Verification & Results)
設定変更後にOSを再起動し、再度 `sudo journalctl -p err -b` を実行して、NFSおよびBluetoothに起因する不要なエラーログが完全に消失していることを確認しました。

ログのノイズが削減されたことにより、本当に注意を払うべきクリティカルなアラート（ストレージのfsckエラーなど）を迅速に検知しやすい状態へと最適化できました。
