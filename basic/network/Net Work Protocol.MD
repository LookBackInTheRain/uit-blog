# 网络协议

# OSI（Open System Interconnect）七层模型
1. 应用层（为应用程序提供服务）
2. 表示层（数据格式转换，数据加密）
3. 会话层（建立、管理和维护回话）
4. 传输层（建立、管理和维护端到端的连接）
5. 网络层（IP选址和路由选择）
6. 数据链路层（提供介质和链路管理）
7. 物理层


# TCP/IP 四层模型 和 OSI七层模型的对比

<table>
    <th>OSI七层模型</th>
    <th>TCP/IP 四层模型</th>
    <th>协议</th>
    <tbody>
        <tr>
            <td>应用层</td>
            <td rowspan='3'>应用层</td>
            <td rowspan='3'>HTTP、TFTP, FTP, NFS, WAIS<br>SMTP,Telnet, 
            Rlogin, SNMP, Gopher,DNS</td>
        </tr>
        <tr>
            <td>表示层</td>
        </tr>
        <tr>
             <td>会话层</td>
        </tr>
        <tr>
             <td>传输层</td>
             <td>传输层</td>
             <td>TCP, UDP</td>
        </tr>
        <tr>
             <td>网络层</td>
             <td>网络层</td>
             <td>IP, ICMP, ARP, RARP, AKP, UUCP</td>
        </tr>
        <tr>
            <td>数据链路层</td>
            <td rowspan='2'>网络接口层</td>
            <td rowspan='2'>FDDI, Ethernet, Arpanet, PDN, SLIP, PPP,IEEE 802.1A, IEEE 802.2到IEEE 802.1</td>
        </tr>
        <tr>
             <td>物理层</td>
        </tr>
    </tbody>
</table>


