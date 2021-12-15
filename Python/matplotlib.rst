使用matplotlib画图
==============================

一. 安装

.. code-block:: bash
    
    pip install matplotlib
    pip install numpy

二. 画图

画曲线图, 一般需要x轴数据和y轴数据, 两组数据个数要相同; 如果只有一组数据, 就用数据索引作为x轴数据

... code-block:: python

    import os
    import re
    import numpy as np
    import matplotlib.pyplot as plt

    # 获取IP, 当前目录下有几个IP命名的目录, 该目录下存储IP对应的所有日志
    def get_hub_ips():
        hub_ips = []
        for dir_name in os.listdir():
            if dir_name.startswith('192.'):
                hub_ips.append(dir_name)

        return hub_ips

    # 获取目录下ping_checked.log日志文件中的数据, 返回数据列表
    def get_ping_datas(hub_ip):
        pinglog = os.path.join(hub_ip, 'ping_checked.log')
        datas = []
        m_ping = re.compile('time=(\d+[\.\d]+) ms')
        with open(pinglog) as f:
            for line in f:
                m = m_ping.findall(line)
                for d in m:
                    datas.append(float(d))

        return datas

    # 获取目录下下行日志文件中的数据, 返回两个(发端和收端)数据列表
    def get_dl_datas(hub_ip):
        dltxlog = os.path.join(hub_ip, 'iperf_du_dl.log')
        txdatas = []
        m_dl = re.compile('(\d+[\.\d]+) Mbits/sec')
        with open(dltxlog) as f:
            for line in f:
                m = m_dl.findall(line)
                for d in m:
                    txdatas.append(float(d))

        rxdatas = []
        dlrxlog = os.path.join(hub_ip, 'iperf_hub_dl.log')
        with open(dlrxlog) as f:
            for line in f:
                m = m_dl.findall(line)
                for d in m:
                    rxdatas.append(float(d))

        return txdatas, rxdatas

    # 获取目录下上行日志文件中的数据, 返回两个(发端和收端)数据列表
    def get_ul_datas(hub_ip):
        ulrxlog = os.path.join(hub_ip, 'iperf_du_ul.log')
        rxdatas = []
        m_dl = re.compile('(\d+[\.\d]+) Mbits/sec')
        with open(ulrxlog) as f:
            for line in f:
                m = m_dl.findall(line)
                for d in m:
                    rxdatas.append(float(d))

        txdatas = []
        ultxlog = os.path.join(hub_ip, 'iperf_hub_ul.log')
        with open(ultxlog) as f:
            for line in f:
                m = m_dl.findall(line)
                for d in m:
                    txdatas.append(float(d))

        return txdatas, rxdatas

    # 画图并保存
    def draw_chart(pingdatas, dldatas, uldatas, hub_ip):
        txdldatas, rxdldatas = dldatas # 传入的是两个列表
        txuldatas, rxuldatas = uldatas # 传入的是两个列表
        dl_len = min(len(txdldatas), len(rxdldatas))  # 下行数据个数, 取最小值作为x轴数据, 数据多的一个会删除多余的数据
        ul_len = min(len(txuldatas), len(rxuldatas))  # 上行数据个数, 取最小值作为x轴数据, 数据多的一个会删除多余的数据

        plt.figure(figsize=(15, 8), dpi=120)  # 设置图片尺寸, figsize分别设置长和宽, 单位为英寸, 1英寸等于2.5cm, A4纸是21*30cm的纸张; dpi为分辨率, 默认为80, 相同的尺寸, dpi越大, 图片越大
        '''
        其他参数:
        num: 图像编号或名称, 数字为编号, 字符串为名称
        figsize:指定figure的宽和高, 单位为英寸
        dpi: 参数指定绘图对象的分辨率, 即每英寸多少个像素, 缺省值为80
        facecolor: 背景颜色
        edgecolor: 边框颜色
        frameon: 是否显示边框
        '''
        plt.subplots_adjust(left=0.05, right=0.98, top=0.95, bottom=0.05)  # 设置边距
        
        xpoints = np.array(range(dl_len))   # x轴数据
        ytxpoints = np.array(txdldatas[:dl_len])  # y轴数据
        yrxpoints = np.array(rxdldatas[:dl_len])  # y轴数据, 两个y轴数据, 这里一个图标中会画两条线
        plt.subplot2grid((30,1),(0,0),colspan=1,rowspan=8)   # 设置图标在整个图片中的位置
        '''
        第一个元组表示图片一共分成多少行和多少列, 这里将图片分成30行, 1列
        第二个元组表示该图标起始的行和列, 第一个图, 起始行和列都为0
        colspan表示该图表占用多少列, 一共就1列, 占1列, 说明该图表横向占满
        rowspan表示该图表占用多少行, 一共30行, 占8行, 改图表大概占整个图片的上面三分之一
        '''
        plt.title(hub_ip+'_dl') #设置图标标题
        plt.plot(xpoints, ytxpoints, color='red', label='tx', marker='.', markersize=4)  #设置曲线颜色, 图例, 点的形状, 点的大小
        '''
        xpoints: x轴数据
        ytxpoints: y轴数据
        color: 线条颜色
        label: 设置图例
        marker: 设置点的形状
        markersize: 设置点的大小
        '''
        plt.plot(xpoints, yrxpoints, color='green', label='rx', marker='.', markersize=4)
        plt.legend(loc='upper right')  #设置图例位置, 边框, 颜色等属性
        plt.ylabel('Mbits/sec')  # 设置坐标轴单位
        
        xpoints = np.array(range(ul_len))
        ytxpoints = np.array(txuldatas[:ul_len])
        yrxpoints = np.array(rxuldatas[:ul_len])
        plt.subplot2grid((30,1),(11,0),colspan=1,rowspan=8)
        plt.title(hub_ip+'_ul')
        plt.plot(xpoints, ytxpoints, color='red', label='tx', marker='.', markersize=4)
        plt.plot(xpoints, yrxpoints, color='green', label='rx', marker='.', markersize=4)
        plt.legend(loc='upper right')
        plt.ylabel('Mbits/sec')
        
        xpoints = np.array(range(len(pingdatas)))
        ypoints = np.array(pingdatas)
        plt.subplot2grid((30,1),(22,0),colspan=1,rowspan=8)
        plt.title(hub_ip+'_ping')
        plt.plot(xpoints, ypoints, color='red', label='tx', marker='.', markersize=4)
        plt.ylabel('ms')

        # plt.show()
        plt.savefig(os.path.join(hub_ip, '{}.jpg'.format(hub_ip)))  # 保存为jpg文件
        plt.close()


    def main():
        hub_ips = get_hub_ips()
        for hub_ip in hub_ips:
            print(hub_ip)
            pingdatas = get_ping_datas(hub_ip)
            txdldatas, rxdldatas = get_dl_datas(hub_ip)
            txuldatas, rxuldatas = get_ul_datas(hub_ip)

            draw_chart(pingdatas, [txdldatas, rxdldatas], [txuldatas, rxuldatas], hub_ip)

    if __name__ == '__main__':
        main()

最终图片:

.. image:: images/matplotlib/1.jpeg













