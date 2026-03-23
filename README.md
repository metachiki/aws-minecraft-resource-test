# AWS EC2 (t3.micro) 環境における高負荷MODサーバーの稼働限界検証

## 概要
自宅のDocker環境（8GB RAM推奨）からAWS無料利用枠（t3.micro / 1GB RAM）へMinecraft Fabricサーバー（2.2GBデータ）を移設し、リソース制約下でのパフォーマンスと稼働限界を検証した。

## 技術スタック
- **クラウド:** AWS EC2 (Ubuntu 24.04 LTS)
- **コンテナ:** Docker, Docker Compose
- **メモリ管理:** Linux Swap (2GB), JVM Tuning (G1GC)
- **ネットワーク:** AWS Security Groups, MyDNS (DDNS)

## 検証プロセスと課題解決
1. **リソース制約の克服:** 物理メモリ1GBに対し、Linuxスワップ領域を2GB構築し、仮想メモリを3GB相当まで拡張。
2. **JVMチューニング:** `Aikar's Flags` を参考に、ガベージコレクション（G1GC）の最適化を実施。
3. **データ移行:** 2.2GBの巨大なワールドデータを、SSH鍵権限（chmod 600等）を適切に設定し、`scp` を用いてセキュアに転送。

## 検証結果
- **起動フェーズ:** メモリ最適化により、MOD読み込み完了（Done!）まで到達を確認。
- **実運用への考察:** プレイヤー接続時のI/O負荷およびメモリ消費の増大により、サーバーラグ（Can't keep up!）が発生。t3.microの限界値を特定し、実運用には t3.medium（4GB）以上のサイジングが必要であるとの技術的結論を得た。
