# Loxodon Framework Bundle

**AssetBundle Manager for Unity3D**

*Developed by Clark*

Requires Unity 5.3.0 or higher.

Loxodon Framework Bundle is a AssetBundle manager.

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
- AssetBundle 加密
- AssetBundle Build

## Quick start

```C#
    private IResources resources;

    void Awake()
    {
        /* Create a BundleManifestLoader. */
        IBundleManifestLoader manifestLoader = new BundleManifestLoader();

        /* Loads BundleManifest. */
        BundleManifest manifest = manifestLoader.Load(BundleUtil.GetReadOnlyDirectory() 
		+ BundleSetting.ManifestFilename);

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
## Simulation mode in the editor
- In the editor,you can enable the simulation mode of the loading.Loads assets without having to build AssetBundle.

![](Resources/menu.png)

```C#
    void Awake()
    {
#if UNITY_EDITOR
        if (SimulationSetting.IsSimulationMode)
        {
            /* Create a PathInfoParser. */
            //IPathInfoParser pathInfoParser = new SimplePathInfoParser("@");
            IPathInfoParser pathInfoParser = new SimulationAutoMappingPathInfoParser();

            /* Create a BundleManager */
            IBundleManager manager = new SimulationBundleManager();

            /* Create a BundleResources */
            resources = new SimulationResources(pathInfoParser, manager);
        }
#endif
    }

    IEnumerator Start()
    {
        string path = "LoxodonFramework/BundleExamples/Models/Green/Green.prefab";
        IProgressResult<float, GameObject> result = resources.LoadAssetAsync<GameObject>(path);
        while (!result.IsDone)
        {
            Debug.LogFormat("Progress:{0}%", result.Progress * 100);
            yield return null;
        }

        if (result.Exception != null)
            yield break;

        GameObject.Instantiate(result.Result);
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

## Custom ILoaderBuilder

```C#
public class CustomBundleLoaderBuilder : AbstractLoaderBuilder
    {
        private bool useCache;
        private IDecryptor decryptor;

        public CustomBundleLoaderBuilder(Uri baseUri, bool useCache) : this(baseUri, useCache, null)
        {
        }

        public CustomBundleLoaderBuilder(Uri baseUri, bool useCache, IDecryptor decryptor) : base(baseUri)
        {
            this.useCache = useCache;
            this.decryptor = decryptor;
        }

        public override BundleLoader Create(BundleManager manager, BundleInfo bundleInfo, BundleLoader[] dependencies)
        {
            Uri loadBaseUri = this.BaseUri;
            
            if (this.useCache && BundleUtil.ExistsInCache(bundleInfo))
            {
                loadBaseUri = this.BaseUri;
                return new WWWBundleLoader(new Uri(loadBaseUri, bundleInfo.Filename), bundleInfo, dependencies, manager, this.useCache);
            }

            if (BundleUtil.ExistsInStorableDirectory(bundleInfo))
            {
                /* Path: Application.persistentDataPath + "/bundles/" + bundleInfo.Filename  */

                loadBaseUri = new Uri(BundleUtil.GetStorableDirectory());
            }
            else if (BundleUtil.ExistsInReadOnlyDirectory(bundleInfo))
            {
                /* Path: Application.streamingAssetsPath + "/bundles/" + bundleInfo.Filename */

                loadBaseUri = new Uri(BundleUtil.GetReadOnlyDirectory());
            }

            if (bundleInfo.IsEncrypted)
            {
                if (this.decryptor != null && bundleInfo.Encoding.Equals(decryptor.AlgorithmName))
                    return new CryptographBundleLoader(new Uri(loadBaseUri, bundleInfo.Filename), bundleInfo, dependencies, manager, decryptor);

                throw new NotSupportedException(string.Format("Not support the encryption algorithm '{0}'.", bundleInfo.Encoding));
            }

            return new WWWBundleLoader(new Uri(loadBaseUri, bundleInfo.Filename), bundleInfo, dependencies, manager, this.useCache);
        }
    }
```

## Custom BundleLoader

```C#
public class UnityWebRequestBundleLoaderBuilder : AbstractLoaderBuilder
    {
        public UnityWebRequestBundleLoaderBuilder(System.Uri baseUri) : base(baseUri)
        {
        }

        public override BundleLoader Create(BundleManager manager, BundleInfo bundleInfo, BundleLoader[] dependencies)
        {
            return new UnityWebRequestBundleLoader(new System.Uri(this.BaseUri , bundleInfo.Filename), bundleInfo, dependencies, manager);
        }
    }

    public class UnityWebRequestBundleLoader : BundleLoader
    {
        public UnityWebRequestBundleLoader(System.Uri uri, BundleInfo bundleInfo, BundleLoader[] dependencies, BundleManager manager) : base(uri, bundleInfo, dependencies, manager)
        {
        }

        protected override IEnumerator DoLoadAssetBundle(IProgressPromise<float, AssetBundle> promise)
        {
            if (this.BundleInfo.IsEncrypted)
            {
                promise.UpdateProgress(0f);
                promise.SetException(new System.NotSupportedException(string.Format("The data of the AssetBundle named '{0}' is encrypted,use the CryptographBundleLoader to load,please.", this.BundleInfo.Name)));
                yield break;
            }

            string path = this.GetAbsoluteUri();
            using (UnityWebRequest www = UnityWebRequest.GetAssetBundle(path, this.BundleInfo.Hash, this.BundleInfo.CRC))
            {
                www.Send();
                while (!www.isDone)
                {
                    promise.UpdateProgress(www.downloadProgress);
                    yield return null;
                }

                if (!string.IsNullOrEmpty(www.error))
                {
                    promise.SetException(new Exception(string.Format("Failed to load the AssetBundle '{0}' at the address '{1}'.Error:{2}", this.BundleInfo.Name, path, www.error)));
                    yield break;
                }

                DownloadHandlerAssetBundle handler = (DownloadHandlerAssetBundle)www.downloadHandler;
                var assetBundle = handler.assetBundle;
                if (assetBundle == null)
                {
                    promise.SetException(new Exception(string.Format("Failed to load the AssetBundle '{0}' at the address '{1}'.", this.BundleInfo.Name, path)));
                    yield break;
                }

                promise.UpdateProgress(1f);
                promise.SetResult(assetBundle);
            }
        }
    }
```

## Build AssetBundle
Build AssetBundle,you can open a Editor window in menu: Tools/Loxodon/Build AssetBundle

![](Resources/BuildAssetBundle.png)


## Contact Us
Email: [yangpc.china@gmail.com](mailto:yangpc.china@gmail.com) 


