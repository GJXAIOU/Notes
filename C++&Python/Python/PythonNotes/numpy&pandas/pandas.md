# pandas

==可以理解为更加像字典==

```python
#!/usr/bin/env python
# -*- coding:utf-8 -*-
#@Time :2019/3/18 14:22
#Author:GJXAIOU


import pandas as pd
import numpy as np
import matplotlib.pyplot as plt

dates = pd.date_range('20180318',periods=6)
array = pd.DataFrame(np.random.rand(6, 4),index= dates, columns=['a', 'b', 'c' ,'d'])
plt.plot(array)
plt.show()
```


