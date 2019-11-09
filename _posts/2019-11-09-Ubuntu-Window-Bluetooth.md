---
layout: post
title: Continuous bluetooth device setting in linux - windows dual booting system
date: 2019-11-09 12:00:00 +0900
author: jihwan.kim
categories: Ubuntu
tags: [Bluetooth]
---

## Continuous bluetooth device setting in linux - windows dual booting system

### 사용환경

Category | Detail
---------|-------
Windows | Windows 10 Pro
Linux | Ubuntu 18.04 LTS
Computer | Lenovo Thinkpad T490

### Introduction

리눅스를 배우기 위해 윈도우/우분투의 Dual OS 시스템을 구축하고 사용하고 있습니다.
두 OS의 목적이 다르다고 해도, 다른 OS로 바꿔서 부팅 할 때마다 사용하고 있던 블루투스 마우스의 페어링을 다시 잡아야 하기 때문에 작업의 연속성이 떨어지게 됩니다.

OS 선택에 관계없이, 블루투스 디바이스의 페어링 기록을 유지할 수 있도록 블루투스 등록 내용을 변조하는 내용에 대해 기록합니다.

블루투스 페어링 정보는 윈도우에서는 Registry에 저장되는 내용으로, 이를 변조하는 것 보다는 우분투 파일시스템에 기록된 페어링 정보를 조작하는 것이 간편합니다.

### 순서

1. 사용하고자 하는 블루투스 디바이스를 Ubuntu에서 페어링
1. Windows 로 재부팅 후, 블루투스 디바이스를 Windows 에서 페어링
    * 페어링 전/후 블루투스 장비의 전원을 꺼두는 것이 권장됩니다. (ID가 변경되거나 하는 경우를 막기 위함)

1. 다시 Ubuntu 로 부팅
1. 페어링 정보 확인

    * Ubuntu의 페어링 정보

    ```zsh
    $cd /var/lib/bluetooth/[PC Bluetooth MAC Address]
    $ls
    ```

    ```zsh
    # Result
    00:20:5B:14:D5:07  C6:80:29:8B:8D:96  cache  settings
    ```


    * Windows의 페어링 정보

    ```zsh
    $sudo apt-get install chntpw
    $mount [Windows setup drive] [Mount objective]
    $cd [Mount objective]/Windows/System32/config
    $chntpw -e SYSTEM
    $cd ControlSet001\Services\BTHPORT\Parameters\Keys\[PC Bluetooth MAC Address]
    $ls
        - (...)\BTHPORT\Parameters\Keys\50eb7193eda5> ls
        - Node has 1 subkeys and 2 values
        - key name
        - <c680298b8d97>
        - size     type              value name             [value if type DWORD]
        -     16  3 REG_BINARY         <00205b14d507>
        -     16  3 REG_BINARY         <MasterIRK>
    ```

1. Bluetooth device MAC Address 확인
    > ls 명령어를 통해, 페어링 된 Bluetooth 장치의 갯수만큼의 폴더 및 Registry Key가 확인되면, 각 ID를 대조해서 같은지 확인합니다. 만약 이름이 약간 다르다면, 우분투에 페어링된 디바이스 폴더 이름을 Key name과 동일하게 변경합니다.
    
    ```zsh
    $mv C6:80:29:8B:8D:96 C6:80:29:8B:8D:97
    ```

    * 일반 Bluetooth의 경우
    
        일반 Bluetooth 장치는 Windows 에서 Value name으로 나타납니다(위 00205b14d507).
        해당 Key의 정보를 옮겨줍니다.

        ```zsh
        # Windows Keys (chntpw)
        $hex 00205b14d507
            - Value <00205b14d507> of type REG_BINARY (3), data length 16 [0x10]
            - :00000  20 80 D2 84 AF 07 DC 2C 7B D2 04 BE 21 02 D5 F2  ......,{...!...

        ```

        ```zsh
        # Result
        ```

        ```zsh
        # Ubuntu device list
        $nano 00:20:5B:14:D5:07/info

        # info 파일의 [LinkKey] - Key 부분을 Windows 페어링 키로 교체해줍니다.
            - [LinkKey]
            - Key=2080D284AF07DC2C7BD204BE2102D5F2
            - Type=4
            - PINLength=0
        ```

    * Bluetooth LE (Low Energy)의 경우
        
        Bluetooth LE  장치는 Windows 에서 Subkey로 나타납니다(위 c680298b8d97). 일반 Bluetooth 장비에서처럼 Key값을 바꾸는 것이 아니라, 추가적인 정보를 모두 바꿔 주어야 합니다.

        ```zsh
        # Windows Keys (chntpw)
        $cd c680298b8d97
        $ls
            - Node has 0 subkeys and 11 values
            - size     type              value name             [value if type DWORD]
            -     16  3 REG_BINARY         <LTK>
            -     4  4 REG_DWORD          <KeyLength>               16 [0x10]
            -     8  b REG_QWORD          <ERand>
            -     4  4 REG_DWORD          <EDIV>                 17058 [0x42a2]
            -     16  3 REG_BINARY         <IRK>
            -     8  b REG_QWORD          <Address>
            -     4  4 REG_DWORD          <AddressType>              1 [0x1]
            -     16  3 REG_BINARY         <CSRK>
            -     4  4 REG_DWORD          <OutboundSignCounter>      0 [0x0]
            -     4  4 REG_DWORD          <MasterIRKStatus>          1 [0x1]
            -     4  4 REG_DWORD          <AuthReq>                 45 [0x2d]

        $hex IRK
            - Value <IRK> of type REG_BINARY (3), data length 16 [0x10]
            - :00000  D9 E0 B0 34 51 14 2B 9D 24 2F 03 8F A5 85 A3 C7 ...4Q.+.$/......

        $hex LTK
            - Value <LTK> of type REG_BINARY (3), data length 16 [0x10]
            - :00000  75 E2 91 7A A9 F9 70 83 D3 C0 EA 04 CB 44 83 AA u..z..p......D..

        $hex CSRK
            - Value <CSRK> of type REG_BINARY (3), data length 16 [0x10]
            - :00000  92 43 DD 9F D6 1E 76 E6 4A EF FF 2D 42 BB 3E CC .C....v.J..-B.>.

        $hex ERand
            - Value <ERand> of type REG_QWORD (b), data length 8 [0x8]
            - :00000  CA 25 8F F3 8B 40 87 25                         .%...@.%
        ```

        각 hex 명령을 통해 얻은 값을 info에 업데이트 해 줍니다. 각각에 대응하는 값은 아래 테이블과 같습니다.

        | Windows Key | Ubuntu info | Method
        -|-------------|-------------|-------
        | IRK | IdentityResolvingKey(Key) | hex
        | LTK | LongTermKey(Key) | hex
        | CSRK | LocalSignatureKey(Key) | hex
        | ERand | LongTermKey(Rand) | **reverse hex, and convert to decimal**
        | EDiv | LongTermKey(Ediv) | decimal (refer to ls value)
        | KeyLength | LongTermKey(EncSize) | decimal (refer to ls value)

        #### 주의사항 : 다른 값과 다르게, ERand는 입력 방법이 복잡합니다.
        ```zsh
        # Original value
        CA 25 8F F3 8B 40 87 25
        # Reversed
        25 87 40 8B F3 8F 25 CA
        # Hex to decimal
        $echo $((0x2587408BF38F25CA))
        # Result
        2704201071090148810
        ```

        ```zsh
        #info result
            - [IdentityResolvingKey]
            - Key=C7A385A58F032F249D2B145134B0E0D9
            - [LocalSignatureKey]
            - Key=9243DD9FD61E76E64AEFFF2D42BB3ECC
            - Counter=0
            - Authenticated=false
            - [LongTermKey]
            - Key=75E2917AA9F97083D3C0EA04CB4483AA
            - Authenticated=0
            - EncSize=16
            - EDiv=17058
            - Rand=2704201071090148810
            - [DeviceID]
            - Source=2
            - Vendor=1133
            - Product=45083
            - Version=17
        ```

설정이 정상적으로 되었으면, 어떤 OS 를 선택하여 부팅 하더라도 정상적으로 페어링이 되는 것을 확인 할 수 있습니다.