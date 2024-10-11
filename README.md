Bu script, bir Azure Storage Account (Depolama Hesabı) oluşturma, yönetme ve özelliklerini değiştirme işlemlerini gerçekleştiren bir Bash betiğidir. Ayrıca, kullanıcı etkileşimi ve hataların ele alınması ile iş akışını daha kullanıcı dostu hale getirir. Şimdi adım adım detaylı olarak açıklayalım:
1. Zaman Takibi Başlatma

bash

start_time=$(date +%s)

  start_time değişkeni, scriptin çalışmaya başladığı zamanı kaydeder. Bu, scriptin sonunda toplam çalışma süresini hesaplamak için kullanılır.

2. Renkli Çıktı Yardımcı Fonksiyonları

bash

function print_info() {
  echo -e "\033[1;34m[INFO]\033[0m $1"
}

function print_success() {
  echo -e "\033[1;32m[SUCCESS]\033[0m $1"
}

function print_error() {
  echo -e "\033[1;31m[ERROR]\033[0m $1"
}

   Bu üç fonksiyon, kullanıcıya bilgi mesajlarını renkli olarak göstermek için tanımlanmıştır:
        print_info(): Bilgi mesajlarını mavi renkte gösterir.
        print_success(): Başarılı işlemleri yeşil renkte gösterir.
        print_error(): Hataları kırmızı renkte gösterir.

3. Scriptin Hata Durumunda Durdurulması

bash

set -e

  set -e komutu, herhangi bir komut hata aldığında scriptin durdurulmasını sağlar. Böylece scriptin ilerideki adımlarda hatalı çalışmasını önler.

4. Kullanıcıdan Dinamik Girdiler Alınması

bash

read -p "Resource Group Name: " resourceGroupName
read -p "Location (e.g., EastUS, WestEurope): " location
read -p "Storage Account Name: " storageAccountName
read -p "SKU (e.g., Standard_LRS, Premium_LRS): " sku
read -p "Storage Account Kind (Storage, StorageV2, BlobStorage): " kind
read -p "Access Tier (Hot, Cool, Archive): " accessTier

  Script, kullanıcıdan kaynak grubu adı, konum, depolama hesabı adı, SKU, tür ve erişim katmanı bilgilerini ister. Bu bilgiler, Azure Storage Account oluşturma ve yapılandırma işlemleri için kullanılır.

5. Depolama Hesabı Adının Otomatik Oluşturulması (Boş Bırakılırsa)

bash

if [ -z "$storageAccountName" ]; then
  storageAccountName="mystorageaccount$(date +%s)"
  print_info "Depolama hesabı adı boş bırakıldı, benzersiz bir ad üretildi: $storageAccountName"
fi

  Eğer kullanıcı depolama hesabı adını boş bırakırsa, script tarih ve saat bilgisini kullanarak benzersiz bir ad üretir. Örneğin: mystorageaccount1622476123.

6. Resource Group Kontrolü ve Oluşturulması

bash

if ! az group show --name $resourceGroupName &>/dev/null; then
  print_info "Resource Group '$resourceGroupName' mevcut değil. Oluşturuluyor..."
  az group create --name $resourceGroupName --location $location && \
  print_success "Resource Group başarıyla oluşturuldu." || \
  { print_error "Hata: Resource Group oluşturulamadı!"; exit 1; }
else
  print_info "Resource Group '$resourceGroupName' zaten mevcut."
fi

  Script, belirtilen resource group'un (kaynak grubu) mevcut olup olmadığını kontrol eder. Eğer mevcut değilse, kullanıcıdan alınan bilgilerle yeni bir resource group oluşturur. Başarı ya da hata durumunda uygun mesajları gösterir.

7. Storage Account Kontrolü ve Oluşturulması

bash

if ! az storage account show --name $storageAccountName --resource-group $resourceGroupName &>/dev/null; then
  print_info "Storage Account '$storageAccountName' oluşturuluyor..."
  az storage account create \
    --name $storageAccountName \
    --resource-group $resourceGroupName \
    --location $location \
    --sku $sku \
    --kind $kind \
    --access-tier $accessTier && \
  print_success "Azure Storage Account başarıyla oluşturuldu: $storageAccountName" || \
  { print_error "Hata: Storage Account oluşturulamadı!"; exit 1; }
else
  print_info "Storage Account '$storageAccountName' zaten mevcut."
fi

  Depolama hesabının belirtilen ad ve kaynak grubunda mevcut olup olmadığını kontrol eder. Eğer depolama hesabı yoksa, kullanıcı girdilerine göre yeni bir depolama hesabı oluşturur. Mevcut ise, bu bilgiyi kullanıcıya bildirir.

8. Storage Account Bilgilerinin Getirilmesi

bash

print_info "Depolama Hesabı Detayları getiriliyor..."
az storage account show --name $storageAccountName --resource-group $resourceGroupName --query "{Name:name, Location:primaryLocation, SKU:sku.name, AccessTier:accessTier}" -o table

  Oluşturulan veya mevcut olan depolama hesabının adını, konumunu, SKU bilgisini ve erişim katmanını tablo formatında gösterir.

9. Erişim Katmanını Değiştirme Seçeneği

bash

read -p "Erişim katmanını değiştirmek ister misiniz? (y/n): " changeTier
if [[ "$changeTier" == "y" ]]; then
  read -p "Yeni erişim katmanı (Hot, Cool, Archive): " newAccessTier
  print_info "Erişim katmanı değiştiriliyor..."
  az storage account update \
    --name $storageAccountName \
    --resource-group $resourceGroupName \
    --access-tier $newAccessTier && \
  print_success "Erişim katmanı başarıyla değiştirildi: $newAccessTier" || \
  { print_error "Hata: Erişim katmanı değiştirilemedi!"; exit 1; }
fi

  Kullanıcıya depolama hesabının erişim katmanını değiştirip değiştirmek istemediği sorulur. Eğer "y" cevabı verilirse, yeni bir erişim katmanı alınarak güncellenir.

10. Zaman Takibi ve Geçen Sürenin Hesaplanması

bash

end_time=$(date +%s)
execution_time=$((end_time - start_time))
print_success "İşlem tamamlandı!"
echo -e "\033[1;36mToplam Geçen Süre: $execution_time saniye.\033[0m"

  Script tamamlandığında, baştaki ve sondaki zaman bilgileri arasındaki fark hesaplanarak toplam çalışma süresi kullanıcıya gösterilir.

Genel İş Akışı

  Kullanıcı girdileri alınır.
    Resource Group ve Storage Account kontrol edilir ve gerekirse oluşturulur.
    Depolama hesabı detayları görüntülenir.
    Erişim katmanını değiştirme seçeneği sunulur.
    İşlem süresi hesaplanıp gösterilir.
