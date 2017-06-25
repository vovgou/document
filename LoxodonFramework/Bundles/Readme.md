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

## Contact Us
Email: [yangpc.china@gmail.com](mailto:yangpc.china@gmail.com) 


