using System;
using System.Collections.Generic;
using System.IO;
using System.Text.Json;
using System.Xml.Serialization;

// Temel sınıf
public abstract class BaseClass
{
    public int Id { get; set; }
    public string Ad { get; set; }
    public string Soyad { get; set; }
    public abstract void BilgiGoster();
}

// Interface
public interface IPerson
{
    void GirisYap();
}

// Öğrenci sınıfı
public class Ogrenci : BaseClass, IPerson
{
    public string OgrenciNo { get; set; }
    public override void BilgiGoster()
    {
        Console.WriteLine($"Öğrenci: {Ad} {Soyad}, No: {OgrenciNo}");
    }

    public void GirisYap()
    {
        Console.WriteLine("Öğrenci girişi yapıldı.");
    }
}

// Öğretim Görevlisi sınıfı
public class OgretimGorevlisi : BaseClass, IPerson
{
    public string Unvan { get; set; }
    public override void BilgiGoster()
    {
        Console.WriteLine($"Öğretim Görevlisi: {Unvan} {Ad} {Soyad}");
    }

    public void GirisYap()
    {
        Console.WriteLine("Öğretim görevlisi girişi yapıldı.");
    }
}

// Ders sınıfı
public class Ders
{
    public string DersAdi { get; set; }
    public int Kredi { get; set; }
    public OgretimGorevlisi Ogretmen { get; set; }
    public List<Ogrenci> Ogrenciler { get; set; } = new List<Ogrenci>();

    public void DersBilgisiGoster()
    {
        Console.WriteLine($"Ders: {DersAdi}, Kredi: {Kredi}");
        Console.WriteLine($"Öğretim Görevlisi: {Ogretmen.Ad} {Ogretmen.Soyad}");
        Console.WriteLine("Kayıtlı Öğrenciler:");
        foreach (var ogrenci in Ogrenciler)
        {
            Console.WriteLine($"- {ogrenci.Ad} {ogrenci.Soyad}");
        }
    }
}

// Dosya yönetimi sınıfı
public static class DosyaYonetimi
{
    public static void JsonKaydet<T>(string dosyaAdi, T veri)
    {
        var json = JsonSerializer.Serialize(veri);
        File.WriteAllText(dosyaAdi, json);
    }

    public static T JsonOku<T>(string dosyaAdi)
    {
        if (!File.Exists(dosyaAdi)) return default;
        var json = File.ReadAllText(dosyaAdi);
        return JsonSerializer.Deserialize<T>(json);
    }

    public static void XmlKaydet<T>(string dosyaAdi, T veri)
    {
        using (var writer = new StreamWriter(dosyaAdi))
        {
            var serializer = new XmlSerializer(typeof(T));
            serializer.Serialize(writer, veri);
        }
    }

    public static T XmlOku<T>(string dosyaAdi)
    {
        if (!File.Exists(dosyaAdi)) return default;
        using (var reader = new StreamReader(dosyaAdi))
        {
            var serializer = new XmlSerializer(typeof(T));
            return (T)serializer.Deserialize(reader);
        }
    }
}

// Program sınıfı
class Program
{
    static void Main()
    {
        // Örnek veri oluşturma
        var ogrenci1 = new Ogrenci { Id = 1, Ad = "Ahmet", Soyad = "Yılmaz", OgrenciNo = "202301" };
        var ogrenci2 = new Ogrenci { Id = 2, Ad = "Ayşe", Soyad = "Demir", OgrenciNo = "202302" };
        var ogretmen = new OgretimGorevlisi { Id = 1, Ad = "Mehmet", Soyad = "Kaya", Unvan = "Prof. Dr." };

        var ders = new Ders { DersAdi = "Matematik", Kredi = 4, Ogretmen = ogretmen };
        ders.Ogrenciler.Add(ogrenci1);
        ders.Ogrenciler.Add(ogrenci2);

        // Verileri kaydetme
        DosyaYonetimi.JsonKaydet("ogrenciler.json", new[] { ogrenci1, ogrenci2 });
        DosyaYonetimi.XmlKaydet("dersler.xml", ders);

        // Verileri okuma
        var kayitliDers = DosyaYonetimi.XmlOku<Ders>("dersler.xml");
        kayitliDers?.DersBilgisiGoster();
    }
}
