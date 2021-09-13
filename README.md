# StorageSolutionProvider

针对不同Android版本的存储策略，提供统一的文件操作接口

* [1.	常用操作及代码示例](#part-1)

    * [1-1.	权限相关](#part-1-1)

		* [1-1-1.	检查文件权限](#part-1-1-1)

		* [1-1-2.	检查文件夹权限](#part-1-1-2)

		* [1-1-3.	申请权限](#part-1-1-3)

		* [1-1-4.	抛出缺少权限异常并捕获](#part-1-1-4)

	* [1-2.	文件操作](#part-1-2)

        * [1-2-1.	读取文件](#part-1-2-1)

        * [1-2-2.	创建文件](#part-1-2-2)

        * [1-2-3.	删除文件](#part-1-2-3)

        * [1-2-4.	修改文件](#part-1-2-4)

		* [1-2-5.	追加文件](#part-1-2-5)

    * [1-3.	文件夹操作](#part-1-3)

        * [1-3-1.	查询文件夹](#part-1-3-1)

        * [1-3-2.	创建文件夹](#part-1-3-2)

        * [1-3-3.	删除/更改文件夹](#part-1-3-3)

* [2.	更新日志](#part-2)

	* [2-1.	version 0.0.1](#part-2-1)

	* [2-2.	version 0.0.2](#part-2-2)

	* [2-3.	version 0.0.3](#part-2-3)

	* [2-4.	version 0.0.4](#part-2-4)

## <span id="part-1">1.常用操作及代码示例</span>

### <span id="part-1-1">1-1.权限相关</span>

#### <span id="part-1-1-1">1-1-1.检查文件权限</span>

我们将文件权限分为五类，分别对应对文件的增删改查以及追加操作

	enum Permission {
		// 文件权限
        QUERY,
        INSERT,
        DELETE,
        UPDATE,
        APPEND,
		...
	}

`Permission.QUERY`:	检查应用是否有查询给定目录文件是否存在/读取文件信息的权限

`Permission.INSERT`:	检查应用是否有创建给定目录文件的权限

`Permission.DELETE`:	检查应用是否有删除给定目录文件的权限

`Permission.UPDATE`：检查应用是否有替换/重命名/写入给定目录文件的权限

`Permission.APPEND`:	在R及R之后的版本，该操作允许应用在一个已存在的媒体文件的文件夹下创建关联的媒体文件，R之前的版本，该操作权限同INSERT

示例1： 查询应用是否有删除`DCIM/Camera`文件夹下名为`test.jpg`文件的权限

	final String filePath = "/storage/emulated/0/DCIM/Camera/test.jpg";
	final IStoragePermissionStrategy.PermissionResult result = StorageSolutionProvider.get().checkPermission(
		filePath,
		IStoragePermissionStrategy.Permission.DELETE
	);

示例2： 查询应用是否有删除`cloud`表中`_id`为`69,96`的文件的权限

	final List<IStoragePermissionStrategy.PermissionResult> results = StorageSolutionProvider.get().checkPermission(
		Lists.newArrayList(69, 96),
		GalleryFilePathResolver.TYPE_CLOUD, // 其它类型需自行拓展
		IStoragePermissionStrategy.Permission.DELETE
	);

`checkPermission()`方法返回的`PermissionResult`对象结构如下：

    class PermissionResult {
        // 检查权限的路径
        public String path;
        // 检查何种权限
        public Permission type;

        // 是否拥有对给定路径的给定权限
        public boolean granted = false;
        // 是否为可申请的权限
        public boolean applicable = false;

        public PermissionResult(String path, Permission type) {
            this.path = path;
            this.type = type;
        }
    }

拿到`PermissionResult`后，接下来需要根据不同的权限情况导向不同的应用行为：

	if (!result.granted) {
		if (result.applicable) {
			// TODO: 申请权限
		} else {
			// TODO: 抛出异常，应用不应有此文件的该项权限
		}
	} else {
		// TODO: 预期的文件操作
	}

#### <span id="part-1-1-2">1-1-2.检查文件夹权限</span>

过去我们使用`FileAPI`时，`(DirFile).canWrite()`方法返回`true`通常代表着可修改文件夹本身及其下文件。为了不造成语义不明的问题，这里将权限显性的拆分为for File & for DIRECTORY两部分，分别代表对文件夹本身的操作权限和对文件的操作权限。

	enum Permission {
		// 对文件的操作
		...
        // 对文件夹本身的操作
        QUERY_DIRECTORY,
        INSERT_DIRECTORY,
        DELETE_DIRECTORY,
        UPDATE_DIRECTORY,
    }

`Permission.QUERY_DIRECTORY`:	检查应用是否有查询给定目录是否存在/读取文件夹信息的权限

`Permission.INSERT_DIRECTORY`:	检查应用是否有创建给定目录的权限

`Permission.DELETE_DIRECTORY`:	除app-specific的目录都会返回false

`Permission.UPDATE_DIRECTORY`:	除app-specific的目录都会返回false

#### <span id="part-1-1-3">1-1-3.申请权限</span>

通过检查权限时获取的`PermissionResult`对象，我们可以得知当前的权限情况，当`!granted && applicable`时，我们可以向用户申请权限。

示例1：默认调起引导UI界面，用户确定后调起[SAF](https://developer.android.google.cn/guide/topics/providers/document-provider?hl=en)授权页面

	StorageSolutionProvider.get().requestPermission(fragmentActivity, result.path, result.type);

示例2：直接跳转到[SAF](https://developer.android.google.cn/guide/topics/providers/document-provider?hl=en)授权页面，略过引导UI界面

	Map<String, Object> params = new HashMap<>();
    params.put(SAFPermissionRequestContract.TYPE, SAFPermissionRequestContract.TYPE_WITHOUT_GUIDE_UI);
	StorageSolutionProvider.get().requestPermission(fragmentActivity, result.path, params, result.type);

#### <span id="part-1-1-4">1-1-4.抛出缺少权限异常并捕获</span>

对相册项目，涉及文件操作的部分通常会开启AsyncTask执行。对较复杂的逻辑，在真正执行文件操作前，往往需要较为繁琐的步骤去获取目标文件名及对应的操作行为。

为了避免检查权限和执行文件操作时copy两份代码来获得必要信息，引入一个名为StoragePermissionMissingException的异常。

    public class StoragePermissionMissingException extends StorageException implements IExpectedException, Parcelable {
	    private final List<IStoragePermissionStrategy.PermissionResult> mPermissionResultList;

		public StoragePermissionMissingException(@NonNull List<IStoragePermissionStrategy.PermissionResult> list) {
            ...
        }

		...

		public void offer(FragmentActivity activity) {
		    ...
		}
	}

在文件操作前检查权限，若缺少必要权限则向调用方抛出该异常。调用方调用`offer()`方法以申请权限。如此则可以避免拷贝两份代码的问题。

### <span id="part-1-2">1-2.文件操作</span>

权限检查通过后，可以通过文件/文件夹的绝对路径 + 预期操作权限，获取`DocumentFile`对象，完成文件操作。

[DocumentFile | Android Developers](https://developer.android.google.cn/reference/kotlin/androidx/documentfile/provider/DocumentFile?hl=en)

同时`StorageSolutionProvider`提供了一些常用文件操作的接口，如

`copyFile(String, String): Boolean` 复制文件

`moveFile(String, String): Boolean` 移动文件

`apply(DocumentFile): Boolean` 触发媒体库异步扫描，将文件改动同步到媒体库

	Note:
	因为以下方式导致的文件变动，无需调用方手动将改动同步到媒体库
	1.调用StorageSolutionProvider提供的方法(直接操作文件描述符和流除外)
	2.操作StorageSolutionProvider返回的DocumentFile对象

`setLastModified(DocumentFile, Long): Boolean` 设置文件/文件夹最后更新时间

#### <span id="part-1-2-1">1-2-1.读取文件</span>

示例: 检查`cloud`表中`_id`为`75`的文件是否存在，读取文件长度

	final IStoragePermissionStrategy.PermissionResult result = StorageSolutionProvider.get().checkPermission(
    	75,
    	GalleryFilePathResolver.TYPE_CLOUD,
		IStoragePermissionStrategy.Permission.QUERY
	).get(0);

	if (!result.granted) {
		if (result.applicable) {
        	// TODO: 申请权限
    	} else {
        	// TODO: 抛出异常，应用不应有此文件的该项权限
		}
	} else {
		DocumentFile file = StorageSolutionProvider.get().getDocumentFile(
			result.path,
			IStoragePermissionStrategy.Permission.QUERY
		);
		if (file == null || !file.exists()) {
			return -1L;
		}
		return file.length();
	}

#### <span id="part-1-2-2">1-2-2.创建文件</span>

当调用`getDocumentFile(String, Permission)`方法时，若传入的`Permission`参数为`Permission.INSERT`，则会在返回`DocumentFile`对象的同时创建文件。

#### <span id="part-1-2-3">1-2-3.删除文件</span>

调用`getDocumentFile(String, Permission)`方法，其中传入的`Permission`参数为`Permission.DELETE`获得文件的`DocumentFile`实例，调用`DocumentFile#delete()`方法删除文件。

#### <span id="part-1-2-4">1-2-4.修改文件</span>

示例:	获取`DCIM/Camera`文件夹下名为`test.jpg`的文件的`Uri`/`输入输出流`/`文件描述符`

	final String filePath = "/storage/emulated/0/DCIM/Camera/test.jpg";
	final IStoragePermissionStrategy.PermissionResult result = StorageSolutionProvider.get().checkPermission(
    	filePath,
    	IStoragePermissionStrategy.Permission.UPDATE
	)
	if (!result.granted) {
    	if (result.applicable) {
        	// TODO: 申请权限
    	} else {
        	// TODO: 抛出异常，应用不应有此文件的该项权限
    	}
	} else {
		DocumentFile file = StorageSolutionProvider.get().getDocumentFile(
			filePath,
			IStoragePermissionStrategy.Permission.UPDATE
		);
		if (file == null || !file.exists()) {
			// TODO: 抛出异常，文件不存在
		}
		final Uri uri = file.getUri();

		final InputStream in = null;
		final OutputStream out = null;
		final ParcelFileDescriptor pfd = null;
		try {
			in = StorageSolutionProvider.get().openInputStream(file);
			out = StorageSolutionProvider.get().openOutputStream(file);
			pfd = StorageSolutionProvider.get().openFileDescriptor(file, "rw");
			// TODO 业务逻辑
		} finally {
			// TODO 释放资源
		}
	}

#### <span id="part-1-2-5">1-2-5.追加文件</span>

示例：编辑位于`DCIM/Camera`目录下的`source.jpg`图片，将结果命名为`result.jpg`并生成在同目录下。

	final String source = "storage/emulated/0/DCIM/Camera/source.jpg";

	// ... some edit work

	final String result = "/storage/emulated/0/DCIM/Camera/result.jpg";

	// should check append permission before

	final Bundle bundle = new Bundle();
    bundle.putParcelable(MediaStore.QUERY_ARG_RELATED_URI, MediaStoreUtils.getFileMediaUri(source));
	final DocumentFile resultFile = StorageSolutionProvider.get().getDocumentFile(
		result,
		IStoragePermissionStrategy.Permission.APPEND,
		bundle
	);
	if (resultFile == null || !resultFile.exists()) {
		// append file failed, maybe there is a file with the same name?
	}
	try (OutputStream out = StorageSolutionProvider.get().openOutputStream(resultFile)) {
		// write bitmap to result file
	}

### <span id="part-1-3">1-3.文件夹操作</span>

#### <span id="part-1-3-1">1-3-1.查询文件夹</span>

基本同[1-2-1.	读取文件](#part-1-2-1)，注意权限为`Permission.QUERY_DIRECTORY`.

	Note.
	由于Scoped Storage的限制，使用新版存储策略的应用，其DocumentFile#listFiles方法返回结果不再可信。

#### <span id="part-1-3-2">1-3-2.创建文件夹</span>

基本同[1-2-2.	创建文件](#part-1-2-2)，注意权限为`Permission.INSERT_DIRECTORY`.

#### <span id="part-1-3-3">1-3-3.删除/更改文件夹</span>

除app-specific的文件夹外，文件夹不应被删除/更改

## <span id="part-2">2.更新日志</span>

### <span id="part-2-1">2-1.verison 0.0.1</span>

•	基本功能

### <span id="part-2-2">2-2.verison 0.0.2</span>

•	`PermissionResult`增加当应用对给定路径无权限时，该权限是否可申请的结果

### <span id="part-2-3">2-3.verison 0.0.3</span>

•	允许使用`ContentObserver`监听授权变更情况

### <span id="part-2-4">2-4.verison 0.0.4</span>

•	不再提供`createDirectory`接口，文件夹创建同文件创建方式统一，交由`getDocumentFile(String, Permission)`实现

•	引入`StoragePermissionMissingException`

•	`getDocumentFile`方法返回的`DocumentFile`对象wrap一层，监控相册的文件删除情况