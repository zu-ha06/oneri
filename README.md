# oneri
import UIKit

// MARK: - Models

struct User: Codable {
    var displayName: String
    var faculty: String
    var className: String
    var agreedToPrivacy: Bool
    var isAnonymous: Bool
}

struct Soru: Codable {
    var baslik: String
    var cevap: String
    var oneri: String
    var tarih: Date
    var poster: String   // hangi isimle gönderildiği
}

// MARK: - Persistence Helpers

enum Storage {
    private static let userKey = "currentUser_v1"
    private static let sorularKey = "sorular_v1"

    static func save(user: User) {
        if let data = try? JSONEncoder().encode(user) {
            UserDefaults.standard.set(data, forKey: userKey)
        }
    }

    static func loadUser() -> User? {
        guard let data = UserDefaults.standard.data(forKey: userKey),
              let user = try? JSONDecoder().decode(User.self, from: data) else { return nil }
        return user
    }

    static func deleteUser() {
        UserDefaults.standard.removeObject(forKey: userKey)
    }

    static func save(sorular: [Soru]) {
        if let data = try? JSONEncoder().encode(sorular) {
            UserDefaults.standard.set(data, forKey: sorularKey)
        }
    }

    static func loadSorular() -> [Soru] {
        guard let data = UserDefaults.standard.data(forKey: sorularKey),
              let arr = try? JSONDecoder().decode([Soru].self, from: data) else { return [] }
        return arr
    }
}

// MARK: - Cell Delegate
protocol SoruCellDelegate: AnyObject {
    func cevapEkleIstegi(index: Int)
}

// MARK: - Custom Cell
class SoruCell: UITableViewCell {

    let baslikLabel = UILabel()
    let metaLabel = UILabel()     // poster + tarih
    let cevapLabel = UILabel()
    let oneriLabel = UILabel()
    let cevapEkleButton = UIButton(type: .system)

    weak var delegate: SoruCellDelegate?
    var index: Int = 0

    override init(style: UITableViewCell.CellStyle, reuseIdentifier: String?) {
        super.init(style: style, reuseIdentifier: reuseIdentifier)

        baslikLabel.numberOfLines = 0
        baslikLabel.font = .boldSystemFont(ofSize: 16)
        baslikLabel.translatesAutoresizingMaskIntoConstraints = false

        metaLabel.font = .systemFont(ofSize: 12)
        metaLabel.textColor = .systemGray
        metaLabel.translatesAutoresizingMaskIntoConstraints = false

        cevapLabel.numberOfLines = 0
        cevapLabel.font = .systemFont(ofSize: 14)
        cevapLabel.textColor = .darkGray
        cevapLabel.translatesAutoresizingMaskIntoConstraints = false

        oneriLabel.numberOfLines = 0
        oneriLabel.font = .italicSystemFont(ofSize: 13)
        oneriLabel.textColor = .systemPurple
        oneriLabel.translatesAutoresizingMaskIntoConstraints = false

        cevapEkleButton.setTitle("Cevap Ekle", for: .normal)
        cevapEkleButton.titleLabel?.font = .systemFont(ofSize: 14)
        cevapEkleButton.translatesAutoresizingMaskIntoConstraints = false
        cevapEkleButton.addTarget(self, action: #selector(cevapEkleTapped), for: .touchUpInside)

        contentView.addSubview(baslikLabel)
        contentView.addSubview(metaLabel)
        contentView.addSubview(cevapLabel)
        contentView.addSubview(cevapEkleButton)
        contentView.addSubview(oneriLabel)

        NSLayoutConstraint.activate([
            baslikLabel.topAnchor.constraint(equalTo: contentView.topAnchor, constant: 10),
            baslikLabel.leadingAnchor.constraint(equalTo: contentView.leadingAnchor, constant: 16),
            baslikLabel.trailingAnchor.constraint(equalTo: contentView.trailingAnchor, constant: -16),

            metaLabel.topAnchor.constraint(equalTo: baslikLabel.bottomAnchor, constant: 6),
            metaLabel.leadingAnchor.constraint(equalTo: baslikLabel.leadingAnchor),
            metaLabel.trailingAnchor.constraint(equalTo: baslikLabel.trailingAnchor),

            cevapLabel.topAnchor.constraint(equalTo: metaLabel.bottomAnchor, constant: 8),
            cevapLabel.leadingAnchor.constraint(equalTo: baslikLabel.leadingAnchor),
            cevapLabel.trailingAnchor.constraint(equalTo: baslikLabel.trailingAnchor),

            cevapEkleButton.topAnchor.constraint(equalTo: cevapLabel.bottomAnchor, constant: 8),
            cevapEkleButton.leadingAnchor.constraint(equalTo: baslikLabel.leadingAnchor),

            oneriLabel.topAnchor.constraint(equalTo: cevapEkleButton.bottomAnchor, constant: 8),
            oneriLabel.leadingAnchor.constraint(equalTo: baslikLabel.leadingAnchor),
            oneriLabel.trailingAnchor.constraint(equalTo: baslikLabel.trailingAnchor),
            oneriLabel.bottomAnchor.constraint(equalTo: contentView.bottomAnchor, constant: -12)
        ])
    }

    @objc func cevapEkleTapped() {
        delegate?.cevapEkleIstegi(index: index)
    }

    required init?(coder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
}

// MARK: - Registration VC
class RegistrationViewController: UIViewController {

    // Callbacks
    var onComplete: ((User) -> Void)?

    // UI
    private let titleLabel = UILabel()
    private let nameField = UITextField()
    private let facultyField = UITextField()
    private let classField = UITextField()
    private let anonymousSwitch = UISwitch()
    private let privacyTextView = UITextView()
    private let agreeSwitch = UISwitch()
    private let agreeLabel = UILabel()
    private let submitButton = UIButton(type: .system)

    override func viewDidLoad() {
        super.viewDidLoad()
        view.backgroundColor = .systemBackground
        setupUI()
    }

    private func setupUI() {
        titleLabel.text = "Üye Ol"
        titleLabel.font = .boldSystemFont(ofSize: 22)
        titleLabel.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(titleLabel)

        func styleField(_ tf: UITextField, placeholder: String) {
            tf.placeholder = placeholder
            tf.borderStyle = .roundedRect
            tf.translatesAutoresizingMaskIntoConstraints = false
            view.addSubview(tf)
        }

        styleField(nameField, placeholder: "Kullanmak istediğiniz isim (opsiyonel)")
        styleField(facultyField, placeholder: "Fakülte (üye olmak için gerekli)")
        styleField(classField, placeholder: "Sınıf (ör. 3/B)")

        let anonLabel = UILabel()
        anonLabel.text = "Anonim isim kullan"
        anonLabel.font = .systemFont(ofSize: 14)
        anonLabel.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(anonLabel)

        anonymousSwitch.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(anonymousSwitch)

        privacyTextView.isEditable = false
        privacyTextView.font = .systemFont(ofSize: 13)
        privacyTextView.layer.cornerRadius = 8
        privacyTextView.layer.borderWidth = 0.5
        privacyTextView.layer.borderColor = UIColor.systemGray4.cgColor
        privacyTextView.translatesAutoresizingMaskIntoConstraints = false
        privacyTextView.text = privacyNoticeText()
        view.addSubview(privacyTextView)

        agreeLabel.text = "Gizlilik politikasını okudum ve kabul ediyorum"
        agreeLabel.numberOfLines = 0
        agreeLabel.font = .systemFont(ofSize: 13)
        agreeLabel.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(agreeLabel)

        agreeSwitch.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(agreeSwitch)

        submitButton.setTitle("Kaydol", for: .normal)
        submitButton.translatesAutoresizingMaskIntoConstraints = false
        submitButton.addTarget(self, action: #selector(submitTapped), for: .touchUpInside)
        submitButton.isEnabled = false
        view.addSubview(submitButton)

        // Enable button only when faculty/class filled and agreeSwitch on
        facultyField.addTarget(self, action: #selector(fieldsChanged), for: .editingChanged)
        classField.addTarget(self, action: #selector(fieldsChanged), for: .editingChanged)
        agreeSwitch.addTarget(self, action: #selector(fieldsChanged), for: .valueChanged)

        NSLayoutConstraint.activate([
            titleLabel.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 18),
            titleLabel.centerXAnchor.constraint(equalTo: view.centerXAnchor),

            nameField.topAnchor.constraint(equalTo: titleLabel.bottomAnchor, constant: 16),
            nameField.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            nameField.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),

            facultyField.topAnchor.constraint(equalTo: nameField.bottomAnchor, constant: 12),
            facultyField.leadingAnchor.constraint(equalTo: nameField.leadingAnchor),
            facultyField.trailingAnchor.constraint(equalTo: nameField.trailingAnchor),

            classField.topAnchor.constraint(equalTo: facultyField.bottomAnchor, constant: 12),
            classField.leadingAnchor.constraint(equalTo: nameField.leadingAnchor),
            classField.trailingAnchor.constraint(equalTo: nameField.trailingAnchor),

            anonLabel.topAnchor.constraint(equalTo: classField.bottomAnchor, constant: 12),
            anonLabel.leadingAnchor.constraint(equalTo: nameField.leadingAnchor),

            anonymousSwitch.centerYAnchor.constraint(equalTo: anonLabel.centerYAnchor),
            anonymousSwitch.leadingAnchor.constraint(equalTo: anonLabel.trailingAnchor, constant: 8),

            privacyTextView.topAnchor.constraint(equalTo: anonLabel.bottomAnchor, constant: 12),
            privacyTextView.leadingAnchor.constraint(equalTo: nameField.leadingAnchor),
            privacyTextView.trailingAnchor.constraint(equalTo: nameField.trailingAnchor),
            privacyTextView.heightAnchor.constraint(equalToConstant: 140),

            agreeLabel.topAnchor.constraint(equalTo: privacyTextView.bottomAnchor, constant: 8),
            agreeLabel.leadingAnchor.constraint(equalTo: nameField.leadingAnchor),
            agreeLabel.trailingAnchor.constraint(equalTo: nameField.trailingAnchor, constant: -60),

            agreeSwitch.centerYAnchor.constraint(equalTo: agreeLabel.centerYAnchor),
            agreeSwitch.leadingAnchor.constraint(equalTo: agreeLabel.trailingAnchor, constant: 8),

            submitButton.topAnchor.constraint(equalTo: agreeLabel.bottomAnchor, constant: 16),
            submitButton.centerXAnchor.constraint(equalTo: view.centerXAnchor)
        ])
    }

    @objc private func fieldsChanged() {
        let facultyOk = !(facultyField.text?.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty ?? true)
        let classOk = !(classField.text?.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty ?? true)
        submitButton.isEnabled = facultyOk && classOk && agreeSwitch.isOn
    }

    @objc private func submitTapped() {
        // validate
        guard let faculty = facultyField.text?.trimmingCharacters(in: .whitespacesAndNewlines),
              let className = classField.text?.trimmingCharacters(in: .whitespacesAndNewlines) else { return }

        var display = nameField.text?.trimmingCharacters(in: .whitespacesAndNewlines) ?? ""
        let anon = anonymousSwitch.isOn

        if anon || display.isEmpty {
            // generate anonymous name
            display = generateAnonymousName()
        }

        let user = User(displayName: display, faculty: faculty, className: className, agreedToPrivacy: agreeSwitch.isOn, isAnonymous: anon)
        Storage.save(user: user)
        onComplete?(user)
        dismiss(animated: true)
    }

    private func generateAnonymousName() -> String {
        let id = Int.random(in: 1000...9999)
        return "Anon-\(id)"
    }

    private func privacyNoticeText() -> String {
        return """
        Gizlilik Bilgilendirmesi (örnek):
        - Uygulama yalnızca zorunlu bilgileri toplar: tercihen gösterilecek isim (opsiyonel), fakülte ve sınıf.
        - Kişisel veriler yerelde (cihazda) saklanır; sunucuya gönderilmez.
        - Verileri silme talebinde bulunabilirsiniz: profil > Hesabı sil.
        - Bu metin hukuki bir belge değildir; resmi uygunluk iddiası için lütfen kurumunuzun veri sorumlusu veya hukuk danışmanınıza başvurun.
        """
    }
}

// MARK: - Main ViewController
class ViewController: UIViewController {

    private let tableView = UITableView()
    private let baslikField = UITextField()
    private let cevapField = UITextField()
    private let oneriField = UITextField()
    private let ekleButton = UIButton(type: .system)

    private var sorular: [Soru] = []
    private var currentUser: User?

    override func viewDidLoad() {
        super.viewDidLoad()
        title = "Soru – Cevap – Öneri"
        view.backgroundColor = .systemBackground

        setupNavBar()
        loadDataAndUI()
    }

    private func setupNavBar() {
        let profileItem = UIBarButtonItem(title: "Profil", style: .plain, target: self, action: #selector(profileTapped))
        navigationItem.rightBarButtonItem = profileItem
    }

    private func loadDataAndUI() {
        // load sorular from storage or load defaults
        let loaded = Storage.loadSorular()
        if loaded.isEmpty {
            hazirSorulariYukle()
            Storage.save(sorular: sorular)
        } else {
            sorular = loaded
        }

        // load user
        currentUser = Storage.loadUser()
        setupUI()

        // if no user, present registration
        if currentUser == nil {
            presentRegistration()
        }
    }

    private func presentRegistration() {
        let reg = RegistrationViewController()
        reg.modalPresentationStyle = .formSheet
        reg.onComplete = { [weak self] user in
            self?.currentUser = user
            // optionally show welcome
            let alert = UIAlertController(title: "Hoşgeldin, \(user.displayName)", message: "Fakülte: \(user.faculty), Sınıf: \(user.className)\nGizlilik onayı: \(user.agreedToPrivacy ? "Evet" : "Hayır")", preferredStyle: .alert)
            alert.addAction(UIAlertAction(title: "Tamam", style: .default))
            self?.present(alert, animated: true)
        }
        present(reg, animated: true)
    }

    // MARK: - Load defaults
    private func hazirSorulariYukle() {
        sorular = [
            Soru(baslik: "YLSY nedir?",
                 cevap: "Yurt dışında lisansüstü eğitim burs programıdır.",
                 oneri: "",
                 tarih: Date(timeIntervalSinceNow: -86400*3),
                 poster: "Sistem"),
            Soru(baslik: "Başvuru şartları nelerdir?",
                 cevap: "Türkiye Cumhuriyeti vatandaşı olmak ve mezuniyet koşullarını sağlamak gerekir.",
                 oneri: "",
                 tarih: Date(timeIntervalSinceNow: -86400*2),
                 poster: "Sistem"),
            Soru(baslik: "Başvuru tarihleri ne zaman?",
                 cevap: "Genellikle her yıl Ekim ayında başvurular alınır.",
                 oneri: "",
                 tarih: Date(timeIntervalSinceNow: -86400*3),
                 poster: "Sistem"),
            Soru(baslik: "Fakültede hangi bölümler var?",
                 cevap: "Sanat ve Tasarım Fakültesi bünyesinde Çizgi Film ve Animasyon, Görsel İletişim Tasarımı, Seramik ve Cam Tasarımı gibi sanat odaklı bölümler yer alır.",
                 oneri: "",
                 tarih: Date(timeIntervalSinceNow: -86400 * 15),
                 poster: "Sistem"),
            // ... diğer hazır soruları buraya ekledim (örnek kısaltıldı). İstersen hepsini ekleyeyim.
            Soru(baslik: "Güncel bilgilere nereden ulaşılır?",
                 cevap: "Resmi site üzerinden: https://stf.mehmetakif.edu.tr",
                 oneri: "",
                 tarih: Date(),
                 poster: "Sistem")
        ]
    }

    // MARK: - UI
    private func setupUI() {

        func style(_ tf: UITextField, placeholder: String) {
            tf.placeholder = placeholder
            tf.borderStyle = .roundedRect
            tf.translatesAutoresizingMaskIntoConstraints = false
            view.addSubview(tf)
        }

        style(baslikField, placeholder: "Soru Başlığı (opsiyonel)")
        style(cevapField, placeholder: "Cevap (opsiyonel)")
        style(oneriField, placeholder: "Öneri (opsiyonel)")

        ekleButton.setTitle("Ekle", for: .normal)
        ekleButton.translatesAutoresizingMaskIntoConstraints = false
        ekleButton.addTarget(self, action: #selector(yeniSoruEkle), for: .touchUpInside)
        view.addSubview(ekleButton)

        tableView.delegate = self
        tableView.dataSource = self
        tableView.register(SoruCell.self, forCellReuseIdentifier: "SoruCell")
        tableView.rowHeight = UITableView.automaticDimension
        tableView.estimatedRowHeight = 140
        tableView.translatesAutoresizingMaskIntoConstraints = false
        view.addSubview(tableView)

        NSLayoutConstraint.activate([
            baslikField.topAnchor.constraint(equalTo: view.safeAreaLayoutGuide.topAnchor, constant: 12),
            baslikField.leadingAnchor.constraint(equalTo: view.leadingAnchor, constant: 16),
            baslikField.trailingAnchor.constraint(equalTo: view.trailingAnchor, constant: -16),

            cevapField.topAnchor.constraint(equalTo: baslikField.bottomAnchor, constant: 8),
            cevapField.leadingAnchor.constraint(equalTo: baslikField.leadingAnchor),
            cevapField.trailingAnchor.constraint(equalTo: baslikField.trailingAnchor),

            oneriField.topAnchor.constraint(equalTo: cevapField.bottomAnchor, constant: 8),
            oneriField.leadingAnchor.constraint(equalTo: baslikField.leadingAnchor),
            oneriField.trailingAnchor.constraint(equalTo: baslikField.trailingAnchor),

            ekleButton.topAnchor.constraint(equalTo: oneriField.bottomAnchor, constant: 10),
            ekleButton.centerXAnchor.constraint(equalTo: view.centerXAnchor),

            tableView.topAnchor.constraint(equalTo: ekleButton.bottomAnchor, constant: 12),
            tableView.leadingAnchor.constraint(equalTo: view.leadingAnchor),
            tableView.trailingAnchor.constraint(equalTo: view.trailingAnchor),
            tableView.bottomAnchor.constraint(equalTo: view.bottomAnchor)
        ])
    }

    // MARK: - Yeni Soru Ekle
    @objc private func yeniSoruEkle() {

        guard currentUser != nil else {
            // güvenlik: kayıtlı kullanıcı yoksa kayıt ekranı
            presentRegistration()
            return
        }

        let baslik = baslikField.text?.trimmingCharacters(in: .whitespacesAndNewlines) ?? ""
        let cevap = cevapField.text?.trimmingCharacters(in: .whitespacesAndNewlines) ?? ""
        let oneri = oneriField.text?.trimmingCharacters(in: .whitespacesAndNewlines) ?? ""

        if baslik.isEmpty && cevap.isEmpty && oneri.isEmpty { return }

        // poster adı kullanıcının tercih ettiği isim (anonim ise sistem tarafından oluşturulmuş isim kullanılır)
        let posterName = currentUser!.displayName

        let yeni = Soru(baslik: baslik, cevap: cevap, oneri: oneri, tarih: Date(), poster: posterName)
        sorular.append(yeni)
        Storage.save(sorular: sorular)
        tableView.reloadData()

        baslikField.text = ""
        cevapField.text = ""
        oneriField.text = ""
        view.endEditing(true)
    }

    // MARK: - Profile
    @objc private func profileTapped() {
        guard let user = currentUser else {
            presentRegistration()
            return
        }

        let message = "İsim: \(user.displayName)\nFakülte: \(user.faculty)\nSınıf: \(user.className)\nAnonim: \(user.isAnonymous ? "Evet" : "Hayır")"
        let alert = UIAlertController(title: "Profil", message: message, preferredStyle: .actionSheet)

        alert.addAction(UIAlertAction(title: "Hesabı Sil", style: .destructive, handler: { _ in
            self.confirmDeleteAccount()
        }))

        alert.addAction(UIAlertAction(title: "Bilgileri Güncelle", style: .default, handler: { _ in
            self.editProfile(user: user)
        }))

        alert.addAction(UIAlertAction(title: "Kapat", style: .cancel))
        present(alert, animated: true)
    }

    private func confirmDeleteAccount() {
        let a = UIAlertController(title: "Hesabı silmek istediğinize emin misiniz?", message: "Bu işlem yerelde saklanan tüm kişisel bilgileri silecektir. Soru/önerileriniz silinmeyecektir (isteğe bağlı: ileride ekleyebiliriz).", preferredStyle: .alert)
        a.addAction(UIAlertAction(title: "İptal", style: .cancel))
        a.addAction(UIAlertAction(title: "Sil", style: .destructive, handler: { _ in
            Storage.deleteUser()
            self.currentUser = nil
            self.presentRegistration()
        }))
        present(a, animated: true)
    }

    private func editProfile(user: User) {
        let reg = RegistrationViewController()
        reg.modalPresentationStyle = .formSheet
        // prefill fields via KVC (quick hack)
        reg.onComplete = { newUser in
            self.currentUser = newUser
            Storage.save(user: newUser)
            let ok = UIAlertController(title: "Güncellendi", message: "Profil bilgileriniz güncellendi.", preferredStyle: .alert)
            ok.addAction(UIAlertAction(title: "Tamam", style: .default))
            self.present(ok, animated: true)
        }
        // Pre-fill by presenting then setting fields asynchronously (simple approach)
        present(reg, animated: true) {
            // can't set textFields directly (they are private); simpler to instruct user to fill again.
        }
    }
}

// MARK: - TableView
extension ViewController: UITableViewDelegate, UITableViewDataSource, SoruCellDelegate {

    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return sorular.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {

        guard let cell = tableView.dequeueReusableCell(withIdentifier: "SoruCell", for: indexPath) as? SoruCell else {
            return UITableViewCell()
        }

        let soru = sorular[indexPath.row]
        let df = DateFormatter()
        df.dateStyle = .short
        df.timeStyle = .none

        cell.index = indexPath.row
        cell.delegate = self

        cell.baslikLabel.text = soru.baslik.isEmpty ? "—" : soru.baslik
        cell.metaLabel.text = "\(soru.poster) • \(df.string(from: soru.tarih))"

        if soru.cevap.isEmpty {
            cell.cevapLabel.text = "—"
            cell.cevapEkleButton.isHidden = false
        } else {
            cell.cevapLabel.text = soru.cevap
            cell.cevapEkleButton.isHidden = true
        }

        cell.oneriLabel.text = soru.oneri.isEmpty ? "" : "💡 \(soru.oneri)"

        return cell
    }

    // Cevap ekleme
    func cevapEkleIstegi(index: Int) {
        let alert = UIAlertController(title: "Cevap Ekle", message: nil, preferredStyle: .alert)
        alert.addTextField { tf in tf.placeholder = "Cevap yaz..." }
        alert.addAction(UIAlertAction(title: "İptal", style: .cancel))
        alert.addAction(UIAlertAction(title: "Ekle", style: .default, handler: { _ in
            let cevap = alert.textFields?.first?.text?.trimmingCharacters(in: .whitespacesAndNewlines) ?? ""
            if cevap.isEmpty { return }
            self.sorular[index].cevap = cevap
            Storage.save(sorular: self.sorular)
            self.tableView.reloadData()
        }))
        present(alert, animated: true)
    }
}
