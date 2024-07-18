## 明阳设备端口测序
### 1. MYCNP-SV-P1 (蓝色/1U/Intel 平台)

| 接口位置 | 接口名称 | 接口形态 | 网卡 busPath   | 网卡系统名称  | 网卡型号                            |
| ---- | ---- | ---- | ------------ | ------- | ------------------------------- |
| 板载   | ens1 | 1G电  | 0000:05:00.0 | enp5s0  | I211 Gigabit Network Connection |
| 板载   | ens2 | 1G电  | 0000:06:00.0 | enp6s0  | I211 Gigabit Network Connection |
| 板载   | ens3 | 1G电  | 0000:07:00.0 | enp7s0  | I211 Gigabit Network Connection |
| 板载   | ens4 | 1G电  | 0000:08:00.0 | enp8s0  | I211 Gigabit Network Connection |
| 板载   | ens5 | 1G电  | 0000:0b:00.0 | enp11s0 | I211 Gigabit Network Connection |
| 板载   | ens6 | 1G电  | 0000:0c:00.0 | enp12s0 | I211 Gigabit Network Connection |
| 板载   | ens7 | 1G电  | 0000:12:00.0 | enp18s0 | I211 Gigabit Network Connection |
| 板载   | ens8 | 1G电  | 0000:13:00.0 | enp19s0 | I211 Gigabit Network Connection |
### 2. MYCNP-SV-P2 (蓝色/1U/Intel 平台)

| 接口位置 | 接口名称 | 接口形态 | 网卡 busPath   | 网卡系统名称    | 网卡型号                                  |
| ---- | ---- | ---- | ------------ | --------- | ------------------------------------- |
| 板载   | ens1 | 1G电  | 0000:05:00.0 | enp5s0    | I211 Gigabit Network Connection       |
| 板载   | ens2 | 1G电  | 0000:06:00.0 | enp6s0    | I211 Gigabit Network Connection       |
| 板载   | ens3 | 1G电  | 0000:07:00.0 | enp7s0    | I211 Gigabit Network Connection       |
| 板载   | ens4 | 1G电  | 0000:08:00.0 | enp8s0    | I211 Gigabit Network Connection       |
| 板载   | ens5 | 1G电  | 0000:0b:00.0 | enp11s0   | I211 Gigabit Network Connection       |
| 板载   | ens6 | 1G电  | 0000:0c:00.0 | enp12s0   | I211 Gigabit Network Connection       |
| 板载   | ens7 | 1G电  | 0000:12:00.0 | enp18s0   | I211 Gigabit Network Connection       |
| 板载   | ens8 | 1G电  | 0000:13:00.0 | enp19s0   | I211 Gigabit Network Connection       |
| 板卡A  | GE1  | 1G光  | 0000:19:00.0 | enp25s0f0 | I350 Gigabit Fiber Network Connection |
| 板卡A  | GE2  | 1G光  | 0000:19:00.1 | enp25s0f1 | I350 Gigabit Fiber Network Connection |
| 板卡A  | GE3  | 1G光  | 0000:19:00.2 | enp25s0f2 | I350 Gigabit Fiber Network Connection |
| 板卡A  | GE4  | 1G光  | 0000:19:00.3 | enp25s0f3 | I350 Gigabit Fiber Network Connection |
### 3. MYCNP-SV-P2C (红色/2U/海光平台)

| 接口位置 | 接口名称   | 接口形态 | 网卡 busPath   | 网卡系统名称    | 网卡型号                                       |
| ---- | ------ | ---- | ------------ | --------- | ------------------------------------------ |
| 板载   | ETH0   | 1G电  | 0000:05:00.0 | eno1      | WX1860A2 Gigabit Ethernet Controller       |
| 板载   | ETH1   | 1G电  | 0000:05:00.1 | eno2      | WX1860A2 Gigabit Ethernet Controller       |
| 板载   | ETH2   | 1G电  | 0000:08:00.0 | eno3      | WX1860A4 Gigabit Ethernet Controller       |
| 板载   | ETH3   | 1G电  | 0000:08:00.1 | eno4      | WX1860A4 Gigabit Ethernet Controller       |
| 板载   | ETH4   | 1G电  | 0000:08:00.2 | eno5      | WX1860A4 Gigabit Ethernet Controller       |
| 板载   | ETH5   | 1G电  | 0000:08:00.3 | eno6      | WX1860A4 Gigabit Ethernet Controller       |
| 板载   | SFP1   | 1G光  | 0000:0b:00.0 | eno7      | WX1860A4 Gigabit Ethernet Controller       |
| 板载   | SFP2   | 1G光  | 0000:0b:00.1 | eno8      | WX1860A4 Gigabit Ethernet Controller       |
| 板载   | SFP3   | 1G光  | 0000:0b:00.2 | eno9      | WX1860A4 Gigabit Ethernet Controller       |
| 板载   | SFP4   | 1G光  | 0000:0b:00.3 | eno10     | WX1860A4 Gigabit Ethernet Controller       |
| 板卡A  | GSFP+1 | 10G光 | 0000:0e:00.0 | enp14s0f0 | Ethernet Controller SP1000A for 10GbE SFP+ |
| 板卡A  | GSFP+2 | 10G光 | 0000:0e:00.1 | enp14s0f1 | Ethernet Controller SP1000A for 10GbE SFP+ |
> 备注:
> 1. GSFP+1、GSFP+2 10G光网卡，左侧为ACT指示灯，右侧为Speed指示灯；Speed指示灯亮绿色=速度1Gbps、Speed指示灯亮橙色=速度10Gbps