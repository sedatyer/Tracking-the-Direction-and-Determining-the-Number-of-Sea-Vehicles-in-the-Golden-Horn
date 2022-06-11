Haliç Boğazındaki Deniz Taşıtlarının Yön Takibinin Yapılması ve Sayısının Belirlenmesi

Deniz trafiğinin yoğun olduğu dar boğazlarda, trafiğin kontrol edilebilmesi ve toplam gemi yoğunluğunun saptanabilmesi için belli başlı kontrol sistemleri geliştirilerek; güvenlik önlemleri için çalışmalar yapılmakta ve çevresel diğer etkiler incelenerek rahat bir trafik hizmeti sunulmaya çalışılmaktadır. Bu doğrultuda teknolojinin getirdiği yenilikler göz önüne alınarak görüntü işleme mekanizması ile farklı çalışmalar buna büyük katkı sağlamaktadır. Biz de bu projede belirli alandan kesit alarak görüntüdeki alandan geçen gemi sayısını tespit etmekteyiz. Bunu yaparken hali hazırdaki görüntü belirli orana indirgendikten sonra belirli filtreleme ve algoritmalarla bu sayma işlemini gerçekleştirmekteyiz. Bu proje geliştirilirken Python programlama dili ve Opencv kütüphanesinden yararlanılmıştır. Ölçekleme, eşikleme gibi yöntemler kullanılarak optimal sistem geliştirilmeye çalışılmıştır. 

---

Tracking the Direction and Determining the Number of Sea Vehicles in the Golden Horn

By developing certain control systems in order to control the traffic and to determine the total ship density in the bottlenecks where sea traffic is intense; Studies are carried out for security measures and an effort is made to provide a comfortable traffic service by examining other environmental effects. In this direction, considering the innovations brought by technology, different studies with image processing mechanism make a great contribution to this. In this project, we determine the number of ships passing through the area in the image by taking a cross section from a certain area. While doing this, after the current image is reduced to a certain ratio, we perform this counting with certain filtering and algorithms. While developing this project, Python programming language and Opencv library were used. Using methods such as scaling and thresholding, an optimal system has been developed.


import cv2
import numpy as np

Video_Reader = cv2.VideoCapture("bogaz.mp4")
fgbg = cv2.createBackgroundSubtractorMOG2()                     #ana videoyu kırpma islemi uygulanıyor...

kernel = np.ones((5, 5), np.uint8)                              #5 e 5 lik matrisin icine full 1 koymamızı saglayacak...

class Koordinat:
    def __init__(self, x, y):
        self.x = x
        self.y = y

#karegenislik ve uzunluk maskeleme icin kullanildi...
class Sensor:
    def __init__(self, Koordinat1, Koordinat2, Kare_Genislik, Kare_Uzunluk):
        self.Koordinat1 = Koordinat1
        self.Koordinat2 = Koordinat2
        self.Kare_Genislik = Kare_Genislik
        self.Kare_Uzunluk = Kare_Uzunluk
        self.Maskenin_Alani = abs(self.Koordinat2.x - Koordinat1.x) * abs(self.Koordinat2.y - Koordinat1.y)       #abs mutlak deger islemi uygular...
        self.Maske = np.zeros((Kare_Uzunluk, Kare_Genislik, 1), np.uint8)
        cv2.rectangle(self.Maske, (self.Koordinat1.x, self.Koordinat1.y), (self.Koordinat2.x, self.Koordinat2.y), (255),
                      thickness=cv2.FILLED)
        self.Durum = False
        self.Algilanan_Arac_Sayisi = -1

"""def Golge_Sil(Resim):
    rgb_planes = cv2.split(Resim)
    dst = np.zeros(shape=(5,2))

    result_planes =[]
    result_norm_planes = []
    for plane in rgb_planes:
        dilated_img = cv2.dilate(plane, np.ones((7,7),np.uint8))
        bg_img = cv2.medianBlur(dilated_img,21)
        diff_img = 255 - cv2.absdiff(plane, bg_img)
        norm_img = cv2.normalize(diff_img, dst, alpha=0, beta=255, norm_type=cv2.NORM_MINMAX, dtype=cv2.CV_8UC1)
        result_planes.append(diff_img)
        result_norm_planes.append(norm_img)

    result = cv2.merge(result_planes)
    result_norm = cv2.merge(result_norm_planes)
    return result_norm"""

Sensor1 = Sensor(Koordinat(150, 20), Koordinat(175, 80), 0, 0)
# cv2.imshow("Maske",Sensor1.Maske)

Sensor2 = Sensor(Koordinat(700, 20), Koordinat(725, 220), 0, 0)
# cv2.imshow("Maske",Sensor1.Maske)

font = cv2.FONT_HERSHEY_PLAIN                                                             #yazi tipi türü olan parametre atanir...(FONT_HERSHEY_PLAIN)

while (1):
    ret, Kare = Video_Reader.read()                                 #ret boolean bir fonksiyona cagri yapar. görüntü aliniyorsa true degilse false dondurur
    Kesilmis_Kare = Kare[275:500, 100:1000]                         #Yogun ve net bir şekilde okunabilen kisim için Video boyutlarına kesme islemi uygulandi
    #Kesilmis_Kare = Golge_Sil(Kesilmis_Kare)

    #print(Kesilmis_Kare.shape)

    Arka_Plan_Silinmis_Kare = fgbg.apply(Kesilmis_Kare)
    Arka_Plan_Silinmis_Kare = cv2.morphologyEx(Arka_Plan_Silinmis_Kare, cv2.MORPH_OPEN, kernel)
    ret, Arka_Plan_Silinmis_Kare = cv2.threshold(Arka_Plan_Silinmis_Kare, 80, 255, cv2.THRESH_BINARY)


    #Arka_Plan_Silinmis_Kare = cv2.GaussianBlur(Arka_Plan_Silinmis_Kare,(5,5),0)
    Arka_Plan_Silinmis_Kare = cv2.medianBlur(Arka_Plan_Silinmis_Kare,5)

    contours, hierarchy  = cv2.findContours(Arka_Plan_Silinmis_Kare,cv2.RETR_TREE,cv2.CHAIN_APPROX_NONE)
    Sonuc = Kesilmis_Kare.copy()

    Doldurulmus_Resim = np.zeros((Kesilmis_Kare.shape[0], Kesilmis_Kare.shape[1], 1), np.uint8)           # Burada resim siyahlarla dolduruldu...
    #print(Doldurulmus_Resim.shape)
    for cnt in contours:
        x, y, w, h = cv2.boundingRect(cnt)
        if (w > 20 and h > 10):
            cv2.rectangle(Sonuc, (x, y), (x + w, y + h), (255, 255, 0), thickness=3)                        #kalinlik
            cv2.rectangle(Doldurulmus_Resim, (x, y), (x + w, y + h), (255), thickness=cv2.FILLED)           #255 siyah anlaminda kalinlik da ici dolu olmasi icin bu sekilde yapildi

    cv2.rectangle(Sonuc, (Sensor1.Koordinat1.x, Sensor1.Koordinat1.y), (Sensor1.Koordinat2.x, Sensor1.Koordinat2.y),
                  (0, 0, 255), thickness=cv2.FILLED)

    cv2.rectangle(Sonuc, (Sensor2.Koordinat1.x, Sensor2.Koordinat1.y), (Sensor2.Koordinat2.x, Sensor2.Koordinat2.y),
                  (255, 0, 255), thickness=cv2.FILLED)

    Sensor1_Maske_Sonuc = cv2.bitwise_and(Doldurulmus_Resim, Doldurulmus_Resim, mask=Sensor1.Maske)
    Sensor1_Beyaz_Pixel_Sayisi = np.sum(Sensor1_Maske_Sonuc == 255)             #burada elde edilen beyaz pixel sayisidir....

    #print(Sensor1_Beyaz_Pixel_Sayisi)
    Sensor1_Oran = Sensor1_Beyaz_Pixel_Sayisi / Sensor1.Maskenin_Alani



    Sensor2_Maske_Sonuc = cv2.bitwise_and(Doldurulmus_Resim, Doldurulmus_Resim, mask=Sensor2.Maske)
    Sensor2_Beyaz_Pixel_Sayisi = np.sum(Sensor2_Maske_Sonuc == 255)  # burada elde edilen beyaz pixel sayisidir....

    Sensor2_Oran = Sensor2_Beyaz_Pixel_Sayisi / Sensor2.Maskenin_Alani

    cv2.putText(Sonuc, str("GALATA KOPRUSU"), (350, 35), font, 1,
                (255, 255, 255))

    #print(Sensor1_Oran)
    if (Sensor1_Oran >= 0.50) and Sensor1.Durum == False:
        cv2.rectangle(Sonuc, (Sensor1.Koordinat1.x, Sensor1.Koordinat1.y), (Sensor1.Koordinat2.x, Sensor1.Koordinat2.y),
                      (0, 255, 0), thickness=cv2.FILLED)
        Sensor1.Durum = True
    elif (Sensor1_Oran <= 0.50) and Sensor1.Durum == True:
        cv2.rectangle(Sonuc, (Sensor1.Koordinat1.x, Sensor1.Koordinat1.y), (Sensor1.Koordinat2.x, Sensor1.Koordinat2.y),
                      (255, 0, 255), thickness=cv2.FILLED)
        Sensor1.Durum = False
        Sensor1.Algilanan_Arac_Sayisi += 1
    else:
        cv2.rectangle(Sonuc, (Sensor1.Koordinat1.x, Sensor1.Koordinat1.y), (Sensor1.Koordinat2.x, Sensor1.Koordinat2.y),
                      (0, 0, 255), thickness=cv2.FILLED)

    cv2.putText(Sonuc, str(Sensor1.Algilanan_Arac_Sayisi), (Sensor1.Koordinat1.x + 1, Sensor1.Koordinat1.y + 30), font, 1,
                (255, 255, 255))
    cv2.putText(Sonuc, str("KABATAS"), (Sensor1.Koordinat1.x + 2, Sensor1.Koordinat1.y + 60), font, 1,
                (255, 255, 255))


    #print(Sensor2_Oran)
    if (Sensor2_Oran >= 0.10) and Sensor2.Durum == False:
        cv2.rectangle(Sonuc, (Sensor2.Koordinat1.x, Sensor2.Koordinat1.y), (Sensor2.Koordinat2.x, Sensor2.Koordinat2.y),
                      (0, 255, 0), thickness=cv2.FILLED)
        Sensor2.Durum = True
    elif (Sensor2_Oran <= 0.10) and Sensor2.Durum == True:
        cv2.rectangle(Sonuc, (Sensor2.Koordinat1.x, Sensor2.Koordinat1.y), (Sensor2.Koordinat2.x, Sensor2.Koordinat2.y),
                      (0, 255, 255), thickness=cv2.FILLED)
        Sensor2.Durum = False
        Sensor2.Algilanan_Arac_Sayisi += 1
    else:
        cv2.rectangle(Sonuc, (Sensor2.Koordinat1.x, Sensor2.Koordinat1.y), (Sensor2.Koordinat2.x, Sensor2.Koordinat2.y),
                      (0, 0, 255), thickness=cv2.FILLED)

    cv2.putText(Sonuc, str(Sensor2.Algilanan_Arac_Sayisi), (Sensor2.Koordinat1.x + 1, Sensor2.Koordinat1.y + 90), font, 1,
                (255, 255, 0))
    cv2.putText(Sonuc, str("EMINONU"), (Sensor2.Koordinat1.x + 2, Sensor2.Koordinat1.y + 195), font,1,
                (255, 255, 255))



    cv2.imshow ("Kare",Kare)
    cv2.imshow("Kesilmis_Kare",Kesilmis_Kare)
    cv2.imshow("Arka_Plan_Silinmis_Kare",Arka_Plan_Silinmis_Kare)
    cv2.imshow("Doldurulmus_Resim", Doldurulmus_Resim)
    #cv2.imshow("Sensor1 Maske Sonuc", Sensor1_Maske_Sonuc)
    #cv2.imshow("Sensor2 Maske Sonuc", Sensor2_Maske_Sonuc)
    cv2.imshow("Sonuc", Sonuc)

    k = cv2.waitKey(30) & 0xFF

    if k == 27:
        break

Video_Reader.release()
cv2.destroyAllWindows()
