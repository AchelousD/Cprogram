+----------------------------+
 ffcnn ���������ǰ�������
+----------------------------+

ffcnn ��һ�� c ���Ա�д�ľ��������ǰ�������
ֻ���� 600 ���д����ʵ���������� yolov3��yolo-fastest �����ǰ������
���������κε������⣬�ڱ�׼ c �����¾Ϳ��Ա���ͨ������ VC��msys2+gcc��ubuntu+gcc
�ȶ��ƽ̨�϶�������ȷ�ı�������

������������ darknet��ncnn ��˵�����ܻ�û���κ��Ż�����������Ӽ���׶���������Ϊ
���ѧϰ����������һ���ο�


darknet �� yolov3 ��һЩ�ܽ�
----------------------------
yolov3 ������ṹ���棬ֻ�о���㡢dropout �㡢shortcut �㡢route �㡢maxpool �㡢
upsample ��� yolo ���⼸�����͡����Ҫʵ���������ǱȽ����׵�

����㣺
1. Ҫ�����׾���ĺ���ͼ��㷽��
2. �������� pad��stride �ĺ���
3. ÿ������˻���һ�� bias ������������ÿ�������Ҫ������� bias
   û�й�һ���������batch_normalize��������㷽����
   x += bias;
   x  = activate(x, type);
4. Ҫ������ʲô�Ƿ�����
5. �������ÿ������ĵ㣬��Ҫ���������
6. ���������й�һ��������batch_normalize��������㷽����
   x  = (x + rolling_mean) / sqrt(rolling_variance + 0.00001f)
   x *= scale;
   x += bias;
   x  = activate(x, type);
   ���� rolling_mean��rolling_variance��scale��bias �� darknet �� weights �ļ��п��Զ�ȡ��

dropout �㣺
ǰ������ʱ����һ����Ե��������ڣ��������ݲ����κδ���ֱ�Ӵ�����һ�㼴��

shortcut �㣺
��ָ��������ݺ͵�ǰ���������ӣ�Ȼ�����������һ��

route �㣺
��ָ���Ĳ㣨�������� 4 ������ƴ�ӣ���߲��䣬channel �������ӣ�Ȼ�����������һ��

maxpool �㣺
max �ػ��㣬�� filter ���ǵ�����ȡ���ֵ��Ϊ���

upsample �㣺
�ϲ����㣬�������Ϊ��ͼ��Ŵ�stride ָ���˷Ŵ�����һ��������ڷ��Ϳ�����

yolo �㣺
��һ����Ҫ�Ǹ�������� feature map ����� bbox
�� yolo-fastest Ϊ�����ܹ������� yolo �㣬������ֱ��� 10x10x255 �� 20x20x255
���� 255 ��ʾ�� 255 ��ͨ������ÿ�����ݵĺ������£�
255 = 3 * (4 + 1 + 80)
3 ��ʾ��� grid ������ 3 �� bbox �������
ÿ�� bbox ����������棬4 �� x, y, w, h �������ݣ�1 �� object score ���֣�Ȼ���� 80 �����������
ÿ�� bbox ������ 80 ���������ҳ�������ߵģ���Ϊ��� bbox �ķ��࣬�������С����ֵ��ignore_thresh������
������Ҫ���ȫ�� bbox ����һ���б��棬Ȼ������һ�� nms �������͵õ����ս����

ÿ�� bbox �����ֺ� (x, y, w, h) ���㷽����
�� tx, ty, tw, th, bs �ֱ��Ӧchannel 0, 1, 2, 3, 4 ��ֵ�����滹�� 80 ����������֣�

���ֵļ��㷽����score = sigmoid(bs); ��80 ���������ּ��㷽����һ���ģ�
������㷽����
float bbox_cx = (j + sigmod(tx)) * grid_width;  ��grid_width ������������㼴 0 ��Ŀ�ȳ��Ը�����Ŀ����ÿ�����ӵ����ؿ�ȣ�
float bbox_cy = (i + sigmod(ty)) * grid_height; �������� bbox_cx һ�£�
float bbox_w  = (float)exp(tw) * anchor_box_w;  �����������ϵ����Ҫ�������ϵ����
float bbox_h  = (float)exp(th) * anchor_box_h;  �������� bbox_w  һ�£�

bbox_cx��bbox_cy �����ĵ����꣬bbox_w��bbox_h �ǿ�ߣ�ת��һ�µõ���
x1 = bbox_cx - bbox_w * 0.5f;
y1 = bbox_cy - bbox_h * 0.5f;
x2 = bbox_cx + bbox_w * 0.5f;
y2 = bbox_cy + bbox_h * 0.5f;


darknet �� weights �ļ�
-----------------------

�ļ���ǰ����һ���ļ�ͷ��
#pragma pack(1)
typedef struct {
    int32_t  ver_major, ver_minor, ver_revision;
    uint64_t net_seen;
} WEIGHTS_FILE_HEADER;
#pragma pack()

Ȼ�����ȫ����Ȩ�����ݣ�yolov3��yolo-fastest ��ģ������Ͼ�ֻ�о�����Ȩ�أ���������û��Ȩ�����ݵġ�
ͼ��;���ˣ�filter�������ݶ��� NCHW ��ʽ��filter �����ݴ��˳��Ϊ��

n �� bias
if (batchnorm) {
    n �� scale
    n �� rolling_mean
    n �� rolling_variance
}
n * c * h * w ��Ȩ������


ffcnn ���ص�
------------
1. ��Ϊ����׶��� c ���Դ���ʵ��
2. �����㷨���� 600 ��
3. ���������κε�������
4. ���Ժܷ������ֲ������ƽ̨
5. ����ʱ���Զ��ͷŲ���Ҫ�� layer ��С�ڴ�ռ��
6. �ֽ׶��� make it work first ������ʱ�����Ż�����
7. ֱ��ʹ�� darknet �� .cfg �� .weights �ļ�������Ҫ��ת����


ffcnn vs ncnn ��������
----------------------
ffcnn ���룺https://github.com/rockcarry/ffcnn
ncnn + yolo-fastest ���룺https://github.com/rockcarry/ffyolodet

�������붼��ʹ�õ� yolo-fastest-1.1 ģ�ͣ�����ͼƬ���� test.bmp
�����Լ��� win7 x64 PC + msys2 gcc -O3 ���Խ����
ffcnn: 100 �μ����ʱ��63165 ms  ÿ֡ 632ms  �ڴ�ռ�ã�4.5MB ����
ncnn : 100 �μ����ʱ�� 8659 ms  ÿ֡ 87ms   �ڴ�ռ�ã�45MB  ����

ncnn ���Ǳ� ffcnn ��ܶ࣬����� 7.3 ���������ڴ�ռ�� ffcnn ���˺ܶ�


rockcarry@163.com
20:22 2021/8/7









