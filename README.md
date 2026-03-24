# AWS EC2 (t3.micro) 環境における高負荷MODサーバーの稼働限界検証

## 1. 概要
自宅のDocker環境（8GB RAM推奨）からAWS無料利用枠（t3.micro / 1GB RAM）へMinecraft Fabricサーバー（2.2GBデータ）を移設し、リソース制約下でのパフォーマンスと稼働限界を検証した。

## 2.  技術スタック
- **クラウド:** AWS EC2 (Ubuntu 24.04 LTS)
- **コンテナ:** Docker, Docker Compose
- **メモリ管理:** Linux Swap (2GB), JVM Tuning (G1GC)
- **ネットワーク:** AWS Security Groups, MyDNS (DDNS)

## 3. 検証プロセスと課題解決
1. **リソース制約の克服:** 物理メモリ1GBに対し、Linuxスワップ領域を2GB構築し、仮想メモリを3GB相当まで拡張。
2. **JVMチューニング:** `Aikar's Flags` を参考に、ガベージコレクション（G1GC）の最適化を実施。
3. **データ移行:** 2.2GBの巨大なワールドデータを、SSH鍵権限（chmod 600等）を適切に設定し、`scp` を用いてセキュアに転送。

## 4. 検証結果
- **起動フェーズ:** メモリ最適化により、MOD読み込み完了（Done!）まで到達を確認。
- **実運用への考察:** プレイヤー接続時のI/O負荷およびメモリ消費の増大により、サーバーラグ（Can't keep up!）が発生。t3.microの限界値を特定し、実運用には t3.medium（4GB）以上のサイジングが必要であるとの技術的結論を得た。

## 5. Ansibleによる環境構築の自動化 (Infrastructure as Code)
検証結果を踏まえ、手動で行ったサーバーの初期セットアップ手順をAnsible Playbook (`setup_server.yml`) としてコード化。これにより、新規インスタンスへのデプロイを自動化し、人為的ミスの削減と構築速度の向上を実現した。

### 自動化の対象
- **リソース最適化**: Linux Swap領域（2GB）の自動生成、権限設定、および `/etc/fstab` への永続化。
- **ステータス検証**: `free -h` コマンドを用いたメモリ・スワップ稼働状況の自動取得と出力。

### 実行コマンド
```bash
ansible-playbook setup_server.yml -u ubuntu --private-key=./your-key.pem
\```
