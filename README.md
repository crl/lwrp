# CustomPipeline

由于Unity接口的变化太过频繁,我无法了解旧接口变成一个什么新的接口
所以复制了unity官方最新2019.2P10的轻量级管线的代码来做 自定义

学习unity 算定义管线,可调试版本来做测试

我会改写一些操作 并相应的添加一些注释来做 自我的说明
一些无用的操作 或者文档 我会直接删除掉 以免影响到我对机制的理解


我也会收集一些别的人总结 放入此仓库中 以供更简便的理解



:+1:[预先查看](https://catlikecoding.com/unity/tutorials/scriptable-render-pipeline/) 会帮助你更好的懂此代码

1. 每个物体最多只受4个附加灯光影响
2. 一般主光源会做阴影,但也可为附加光源启用阴影
  - 平形光阴影 (方向等于Matrix.GetColumn(2),w=0) 不需位置但需用正交视角
  - 灯光源阴影 (位置等于Matrix.GetColumn(3),w=1)
  - 聚光源阴影 ...
  - 对它们各自的分级采样
3. HLSL
  - Core.hlsl 为基本的运算操作
  - SpaceTransforms.hlsl 空间转换 及mvp定义
  - Lighting.hlsl 光照的运算，如 顶点光照,球楷模型,lightmap,GI,BlinnPhong,pbr等
  - Shadow.hlsl 用于阴影的计算,包含是否支持屏幕空间阴影,只否多级采样
4. shader里面会include相应的hlsl,如简单的lit.shader(光照) 会引LitForwardPass.hlsl
  - 有点麻烦的是,它对所有平台用了宏,所以看一些写法会比较奇怪,如:tex(_MainTex,uv) 会变成SampleAlbedoAlpha(uv, TEXTURE2D_PARAM(_MainTex, sampler_MainTex))
  - 顶点着色器中会计算最多4盏附加光照,并存储在vertexLight,它使用基本的LightingLambert,还有基本的SH光照
  - 片段中会取主光源,再与gi做MixRealtimeAndBakedGI,做基本的漫反射及高光运算,然后加入最多4盏附加光照的漫反射及高光运算,然后选择性的加入vertexLight
  - 当然片段着色器中 可以选择用哪种形式来做光bdrf,如BlinnPhong/PBR
5. 程序入口LightweightRenderPipeline
  - 收集CameraData,lightData,shadowData
  - rendererSetup没特殊情况下其实为:ForwardRendererSetup 的setup
  - 由上面收集来的数据 来收集或者启用不同的pass
  - 相应该的pass的Setup,会返回是否加入到passExecuters列表中,当然还有一部份是会自动加入基本的流程
  - 是否有postEffect,是否需要copy深度图等
  - ...
  - 对passExecuters 里面的pass做foreach操作,并调用pass.execute();
