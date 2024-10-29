- [Yeni Bir Asset Oluşturmak](#yeni-bir-asset-oluşturmak)
	- [Factory](#factory)
	- [Asset Definition](#asset-definition)
		- [Fonksiyonlar (Override tavsiye edilir)](#fonksiyonlar-override-tavsiye-edilir)
___
# Yeni Bir Asset Oluşturmak
Assetlerin aslında normal UObjectlerden pek bir farkı yoktur. Oyun içinde oluşturulan assetler `GetTransientPackage()` fonksiyonu ile oluşturulurken, Assetler editör tarafından oluşturulan package içerisinde tutulur. Bu assetler kalıcı hafızada tutulur. Bu assetlerin oluşturulması ve yönetilmesini sağlayan 2 ana sınıf vardır: Factory ve AssetDefinition.

## Factory
Bu sınıf objelerin içeriye aktarılması(importing) ve yenilerinin oluşturulmasından sorumludur. Bu sınıf [Factory Detayları](FactoryDetayları.md) yazısında detaylı bir şekilde açıklandı. Bu yazıda sadece temel fonksiyonlardan bahsedeceğim.

Bir factory yeni obje oluşturmak için kullanılacağı zaman genellikle şu şekilde bir constructor kullanılır.
```cpp
UBehaviorTreeFactory::UBehaviorTreeFactory(const FObjectInitializer& ObjectInitializer)
	: Super(ObjectInitializer)
{
	SupportedClass = UBehaviorTree::StaticClass();
	bCreateNew = true;
	bEditAfterNew = true;
}
```
Bu constructor `SupportedClass` ile hangi sınıf için kullanılacağını belirtir. `bCreateNew` ile yeni bir obje oluşturmak için kullanılabileceğini belirtir ve `bEditAfterNew` ile oluşturulduktan sonra düzenlenme ekranına geçmesi gerektiğini belirtir.

```cpp
UObject* UBehaviorTreeFactory::FactoryCreateNew(UClass* Class,UObject* InParent,FName Name,EObjectFlags Flags,UObject* Context,FFeedbackContext* Warn)
{
	check(Class->IsChildOf(UBehaviorTree::StaticClass()));
	return NewObject<UBehaviorTree>(InParent, Class, Name, Flags);;
}
```
Bu fonksiyon ise basitçe yeni obje isteği geldiğinde objeyi oluşturup döndürür.
## Asset Definition
Bu sınıf assetin nasıl bir asset olduğunu, belirli işlemler karşısında ne yapacağını ve özel işlemlerin neler olacağını tanımlar. Eğer tüm fonksiyonları tek tek override etmek istemiyorsanız `UAssetDefinitionDefault` sınıfını üst sınıf olarak kullanabilirsiniz.
### Fonksiyonlar (Override tavsiye edilir)
```cpp
virtual FText GetAssetDisplayName() const override;
```
Bu fonksiyon assetin görünen ismini döndürür
___
```cpp
virtual FLinearColor GetAssetColor() const override;
```
Bu fonksiyon assetin rengini döndürür
___
```cpp
virtual TSoftClassPtr<UObject> GetAssetClass() const override;
```
Bu fonksiyon assetin sınıfını döndürür
___
```cpp
virtual TConstArrayView<FAssetCategoryPath> GetAssetCategories() const override;
```
Bu fonksiyon assetin kategorisini belirler.
```cpp
static FAssetCategoryPath ProjectAssetCategory = FAssetCategoryPath(NSLOCTEXT("ProjectEditor","ProjectBaseCategory","Project"));
static const FAssetCategoryPath Categories[] = {ProjectAssetCategory};
return Categories;
```
şeklinde alt kategoriler kullanılabilir, veya `EAssetCategoryPaths` içerisinden bir kategori seçilip kullanılabilir. 
___
```cpp
virtual EAssetCommandResult OpenAssets(const FAssetOpenArgs& OpenArgs) const override;
```
Bu fonksiyon asset editorlerin açılması için kullanılır. Varsayılan sistem `FSimpleAssetEditor`'ü açar, eğer işinizi görüyorsa bu sistemi bu halde bırakmanızı tavsiye ederim. Detayları [Özel Asset Editorler](OzelAssetEditorler.md) yazısından okuyabilirsiniz.
