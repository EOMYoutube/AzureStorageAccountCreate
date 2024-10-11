Bu Bash betiği, Azure CLI kullanarak bir Azure Resource Group ve Storage Account oluşturur ve yönetir. İşte adım adım açıklaması:

Kullanıcıdan Girdi Alma:

read -p "Resource Group Name: " resourceGroupName: Kullanıcıdan Resource Group adını alır.
read -p "Location (e.g., EastUS, WestEurope): " location: Kullanıcıdan konum bilgisini alır.
read -p "Storage Account Name: " storageAccountName: Kullanıcıdan Storage Account adını alır.
read -p "SKU (e.g., Standard_LRS, Premium_LRS): " sku: Kullanıcıdan SKU bilgisini alır.
read -p "Storage Account Kind (Storage, StorageV2, BlobStorage): " kind: Kullanıcıdan Storage Account türünü alır.
read -p "Access Tier (Hot, Cool, Archive): " accessTier: Kullanıcıdan erişim katmanını alır.
Depolama Hesabı Adı Boşsa Benzersiz Ad Üretme:

if [ -z "$storageAccountName" ]; then: Eğer storageAccountName boşsa,
storageAccountName="mystorageaccount$(date +%s)": mystorageaccount ve zaman damgası ile benzersiz bir ad oluşturur.
Resource Group Kontrolü ve Oluşturma:

if ! az group show --name $resourceGroupName &>/dev/null; then: Eğer belirtilen Resource Group mevcut değilse,
az group create --name $resourceGroupName --location $location: Yeni bir Resource Group oluşturur.
if [ $? -ne 0 ]; then: Eğer Resource Group oluşturma başarısız olursa,
echo "Hata: Resource Group oluşturulamadı!": Hata mesajı verir ve betiği sonlandırır.
else: Eğer Resource Group mevcutsa,
echo "Resource Group '$resourceGroupName' zaten mevcut.": Mevcut olduğunu bildirir.
Storage Account Kontrolü ve Oluşturma:

if ! az storage account show --name $storageAccountName --resource-group $resourceGroupName &>/dev/null; then: Eğer belirtilen Storage Account mevcut değilse,
az storage account create: Yeni bir Storage Account oluşturur.
if [ $? -ne 0 ] ; then: Eğer Storage Account oluşturma başarısız olursa,
echo "Hata: Storage Account oluşturulamadı!": Hata mesajı verir ve betiği sonlandırır.
else: Eğer Storage Account mevcutsa,
echo "Storage Account '$storageAccountName' zaten mevcut.": Mevcut olduğunu bildirir.
Storage Account Bilgilerini Getirme:

az storage account show --name $storageAccountName --resource-group $resourceGroupName --query "{Name:name, Location:primaryLocation, SKU:sku.name, AccessTier:accessTier}" -o table: Storage Account bilgilerini tablo formatında gösterir.
Erişim Katmanını Değiştirme Seçeneği:

read -p "Erişim katmanını değiştirmek ister misiniz? (y/n): " changeTier: Kullanıcıya erişim katmanını değiştirmek isteyip istemediğini sorar.
if [[ "$changeTier" == "y" ]]; then: Eğer kullanıcı "y" cevabını verirse,
read -p "Yeni erişim katmanı (Hot, Cool, Archive): " newAccessTier: Yeni erişim katmanını alır.
az storage account update: Storage Account'un erişim katmanını günceller.
if [ $? -ne 0 ]; then: Eğer güncelleme başarısız olursa,
echo "Hata: Erişim katmanı değiştirilemedi!": Hata mesajı verir ve betiği sonlandırır.
echo "Erişim katmanı başarıyla değiştirildi: $newAccessTier": Başarılı olduğunu bildirir.
İşlem Tamamlandı Mesajı:

echo "İşlem tamamlandı!": Betiğin tamamlandığını bildirir.
