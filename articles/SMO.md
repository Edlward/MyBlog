# 无感FOC滑膜观测器学习
ctime:2020-02-04 20:40:32 +0900|1580816432

标签（空格分隔）： 技术 硬件
---

目标是要通过滑膜观测器来获取电机转子位置

根据电机的数学模型，只要得到A B 相的反电动势即可算出位置

而可以较为简单获取的参数只有相电压(UA UB UC)和相电流(IA IB IC)

我一开始有一个疑问，既然要AB相的反电动势，那么我采出三相的反电动势，在做一个3->2的克拉克变换不就可以得到AB相的反电动势了么？
这个想法实际上是受以前做BLDC6步换向影响，当时6步换向是采集没有导通的相的反电动势，检测是否过零。

但现在在SVPWM中，三相是都导通着，并没有不导通的相，所以并不能检测到三相的反电动势，只能测出相电压。

好了，这是一个小插曲。有了相电压之后，我们还得采样相电流，相电流作何用途等下再讲。

![此处输入图片的描述][1]

从电机方程中，我们可以得到反电动势与电流的方程，方程参数为采样时间、电感、电阻等。
通过不断地调整估算反电动势$E^{s*}$，使估算电流也不断变化。估算电流与实际电流做差，得到差乘以增益再补偿到估算的反电动势中。

这样，当估算电流与实际电流几乎相等时，我们便可以用这个估算的反电动势取计算角度了。

这里有一个误区，由于电机的参数未必准确，因此才采用这种方法。实际上，如果电机的电感电阻等时完全准确的，那么我们直接使用
实际电流套到电机方程中，就可以计算出反电动势了。然而由于电机参数不准确，因此采用这种滑模观测的方法。

需要注意的是，这样得到的反电动势数值上与真实的反电动势一般是不相等的（因为参数未必准确），二者可能差一个系数倍。

但对于计算角度来说，这个无所谓，因为计算角度的时候，是使用反正切，只要alpha相的反电动势与beta相的数量级一致，那么算出的角度，
应该也是可以相信的。

反电动势的公式为：$e^S=w*\phi$
其中，$\phi$ 分为alpha相的磁链和beta相的磁链，需要各自乘以cos和sin

得到两相的反电动势之后，进行反正切运算，就可以得到theta了。


[1]: https://raw.githubusercontent.com/Ncerzzk/MyBlog/master/img/smo.png