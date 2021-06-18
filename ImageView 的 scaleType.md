## center：不放大缩小
1. 原size居中显示
2. 截取图片中间部分同view大小显示

## center_crop：放大
1. 按比例放大，直到两边都等于或者大于view两边
2. 取图片中间显示

## center_inside：大图缩小，小图不变
1. 图片内容完整显示
2. 如果图片太大，按比例缩小原来大小直到两边都小于或者等于view的两边

## fit_center：小图放大、大图缩小，默认的ScaleType
1. 图片内容完整显示
2. 按比例放大缩小直到两边都小于或者等于view的两边，显示在view的中间

## fit_end & fit_start与fit_center类似，但是分别现在在view的顶部和底部

## fit_xy：小图放大，大图缩小
1. 图片内容全部显示
2. 不按比例缩放，只为把view的部分填充满

>效果图见blog：http://blog.csdn.net/larryl2003/article/details/6919513

## matrix：绘图时按照矩阵进行缩放