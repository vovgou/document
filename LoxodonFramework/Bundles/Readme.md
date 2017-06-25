# Loxodon Framework Bundle

**AssetBundle Manager for Unity3D**

*Developed by Clark*

Requires Unity 5.3.0 or higher.

Loxodon Framework Bundle is a lightweight MVVM(Model-View-ViewModel) framework built specifically to target Unity3D.
Databinding and localization are supported.It has a very flexible extensibility.It makes your game development faster and easier.

For tutorials,examples and support,please see the project.You can also discuss the project in the Unity Forums.

The Free version is compatible with MacOSX,Windows,Linux and Android.The Pro version is compatible with MacOSX,Windows,Linux,IOS and Android etc.Check out the Pro version if you want more platforms and full source code.

**Tested in Unity 3D on the following platforms:**  
PC/Mac/Linux  
IOS  
Android

## Downloads  
- [Loxodon Framework Bundle](https://www.assetstore.unity3d.com/#!/content/87419)

## Key Features:
- AssetBundle 依赖关系管理;
- AssetBundle 加载;
- Asset的加载;
- AssetBundle下载;
- AssetBundle 在编辑器模拟加载    

## Quick start

```C#
    private IResources resources;

    void Awake()
    {
        /* Create a BundleManifestLoader. */
        IBundleManifestLoader manifestLoader = new BundleManifestLoader();

        /* Loads BundleManifest. */
        BundleManifest manifest = manifestLoader.Load(BundleUtil.GetReadOnlyDirectory() + BundleSetting.ManifestFilename);

        /* Create a PathInfoParser. */
        IPathInfoParser pathInfoParser = new AutoMappingPathInfoParser(manifest);

        /* Use a custom BundleLoaderBuilder */
        ILoaderBuilder builder = new WWWComplexLoaderBuilder(new Uri(BundleUtil.GetReadOnlyDirectory()), false);

        /* Create a BundleManager */
        IBundleManager manager = new BundleManager(manifest, builder);

        /* Create a BundleResources */
        resources = new BundleResources(pathInfoParser, manager);
    }

    void Start()
    {
        string path = "LoxodonFramework/BundleExamples/Models/Green/Green.prefab";
        IProgressResult<float, GameObject> result = resources.LoadAssetAsync<GameObject>(path);
        result.Callbackable().OnProgressCallback(p =>
        {
            Debug.LogFormat("Progress:{0}%", p * 100);
        });
        result.Callbackable().OnCallback((r) =>
        {
            try
            {
                if (r.Exception != null)
                    throw r.Exception;
                GameObject.Instantiate(r.Result);
            }
            catch (Exception e) 
            { 
                Debug.LogErrorFormat("Error:{0}", e);
            }
        });
    }
```

## PathInfoParser
    PathInfoParser 是将资源的path解析成BundleName和AssetName的工具。我提供了两种类型的PathInfoParser，当然您也可以使用自己的PathInfoParser，只要实现IPathInfoParser接口就可以自定义PathInfoParser。

	注意：所有的AssetName都是相对于根目录Assets的相对路径。如：Assets/Characters/MonkeyKing.prefab的AssetName即为Characters/MonkeyKing.prefab

	a.SimplePathInfoParser

		SimplePathInfoParser支持使用分隔符分隔BundleName和AssetName的方式来组织资源路径。
		示例：

		AssetBundle:characters.unity3d
		Asset:Assets/Characters/MonkeyKing.prefab		
		加载路径:characters@Characters/MonkeyKing.prefab (注：分隔符可以是@，也可以是其他字符)
        
        ```C#

		SimplePathInfoParser parser = new SimplePathInfoParser(new string[]{"@"});	
		resources.LoadAssetAsync<GameObject>("characters@Characters/MonkeyKing.prefab");
        
        ```
		
	b.AutoMappingPathInfoParser
		
		兼顾性能和便捷性，推荐使用AutoMappingPathInfoParser解析器。AutoMappingPathInfoParser会自动创建AssetName和BundleName的映射关系，通过AssetName可以自动找到对应的BundleName。
		如上例子中，
        
		```C#
        
		BundleManifest manifest;
		AutoMappingPathInfoParserparser = new AutoMappingPathInfoParser(manifest);
		resources.LoadAssetAsync<GameObject>("Characters/MonkeyKing.prefab");
        
        ```

## Contact Us
Email: [yangpc.china@gmail.com](mailto:yangpc.china@gmail.com) 


