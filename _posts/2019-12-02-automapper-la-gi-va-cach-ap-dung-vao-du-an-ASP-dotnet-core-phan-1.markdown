---
layout: post
title: AutoMapper là gì và cách áp dụng vào dự án ASP.NET Core - Phần 1
date: 2019-12-02 12:00:30 +0300
image: 16.jpg
tags: [csharp, dotnet, back-end]
---

Rất có thể nhiều lập trình viên đã nghe về khái niệm AutoMapper trong lúc lập trình back-end cho những hệ thống website. Tuy nhiên, lí do tại sao chúng ta nên áp dụng thư viện này trong quá trình thiết kế Web API và những tiện ích thực sự mà nó mang lại thì khá là mơ hồ cho những bạn chưa thực sự áp dụng nó. Do đó, lần này mình sẽ dùng một số trải nghiệm sau một quãng thời gian "lăn lộn" với bộ thư viện này để giúp mọi người hiểu rõ hơn về nó.

Vậy AutoMapper là gì? Theo định nghĩa trên [trang chủ][automapper-docs] của nó thì là "AutoMapper is a simple little library built to solve a deceptively complex problem - getting rid of code that mapped one object to another". E hèm, với trình độ anh văn kha khá của mình thì ý nghĩa của đoạn trên thì thư viện này hỗ trợ map data từ một Object sang 1 Object khác. Đến đây, sẽ có một số người không hiểu, map data là gì, và tại sao phải map data, không phải muốn tạo 1 object mới thì mình khai báo nó bằng object cũ rồi xuất ra là OK sao. Tuy nhiên, map data ở đây không có ý nghĩa là chuyển data y chang từ "đây" sang "kia" mà trong đó còn yêu cầu biến đổi data cho phù hợp.

![]({{site.baseurl}}/img/14.jpg)

Để mình cho một ví dụ cụ thể. Team mình có hai team, chia ra front-end và back-end cùng làm một hệ thống quản lý học sinh. Hệ thống khá là lớn, nhưng ở đây mình chỉ ví dụ vài bảng chính thôi, đó chính là bảng Student và bảng Major. Sau một hồi bàn luận, anh chị em trong team quyết định dùng Microsoft SQL để thiết kế database(mặc dù mình cũng đề xuất dùng MongoDB đi T-T). Ah, dự án này mình dùng ASP.Net Core để lập trình back-end nha.

{% highlight csharp %}
public class Student
{
	public int Id { get; set; }
	public string Code { get; set; }
	public string FirstName { get; set; }
	public string LastName { get; set; }
	public string Address { get; set; }
	public string Email { get; set; }
	public string Phone{ get; set; }
	public string Year { get; set; }
	public DateTime DateOfBirth { get; set; }
	public int MajorId { get; set; }
	public Major Major { get; set; }
}
{% endhighlight %}

{% highlight csharp %}
public class Major
{
	public int Id { get; set; }
	public string Code { get; set; }
	public string Name { get; set; }
}
{% endhighlight %}

Đây chính là hai Model tượng trưng cho hai table Student và Major trong database. Theo thiết kế trên, chúng ta dễ dàng thấy mỗi sinh viên sẽ có một Major(chuyên ngành) duy nhất và xác định. Mội Major lại có một danh sách học sinh để sau này dễ dàng quản lý. Tiếp theo là quá trình dùng Entity Framework để kết nối database, móc dữ liệu để vào Model rồi trả về json thông qua API. Team front-end sẽ lấy data từ những API này để hiển thị lên giao diện cho người dùng có thể truy cập được một cách dễ dàng.

Ok, vậy với một thằng lập trình viên back-end với một vài năm kinh nghiệm, mình nhảy thẳng vào tạo Model, kết nối database, và triển khai API một cách vô cùng trơn tru. Dưới đây là một API đơn giản, trả về danh sách tất cả Student đang được lưu trữ trong hệ thống

{% highlight csharp %}
[HttpGet]
[Route("getallstudent")]
public async Task<IEnumerable<Student>> GetAllStudent()
{
	return await _context.Students
	.Include(s => s.Major)
	.ToArrayAsync();
}
{% endhighlight %}

Kết quả khi gọi API trả về khá là đẹp đẽ. Có thể thấy ngoài thông tin cơ bản của sinh viên, mội Student Object cũng chứa thông tin của Major mà sinh viên đó đang theo học. Nhiêu đây là đủ để cho những bạn front-end hiển thị thông tin lên cho người dùng xem rồi.

{% highlight json %}
[
   {
      "id":1,
      "code":"1331209041",
      "firstName":"Đức Duy",
      "lastName":"Đàm",
      "address":"C57/18, khu 5, Chánh Nghĩa, Bình Dương",
      "email":"duy.dam.k3set@eiu.edu.vn",
      "phone":"09293841597",
      "year":"5",
      "dateOfBirth":"1995-10-18T00:00:00",
      "majorId":1,
      "major":{
         "id":1,
         "code":"CNTT001",
         "name":"Công nghệ thông tin"
      }
   },
   {
      "id":2,
      "code":"1331209042",
      "firstName":"Văn Lang",
      "lastName":"Nguyễn",
      "address":"Chợ Thủ Dầu Một, Bình Dương",
      "email":"hau.tran.k3set@eiu.edu.vn",
      "phone":"09293841597",
      "year":"5",
      "dateOfBirth":"1995-07-18T00:00:00",
      "majorId":1,
      "major":{
         "id":1,
         "code":"CNTT001",
         "name":"Công nghệ thông tin"
      }
   }
]
{% endhighlight %}

Nhưng đời không như là mơ, sau một thời gian phát triển. Một đứa nào đó trong team giao diện nó tìm thấy một bộ thư viện hiển thị danh sách của sinh viên theo dạng bảng khá đẹp và có nhiều tính năng hay ho hoa lá cành. Đưa cho ông leader, xem lướt sơ qua thấy ưng ý, thế là ổng ra lệnh cho team sử dụng bộ thư viện này ngay. Nhưng đời không phải mơ, tuy bộ thư viện này hay nhưng nó lại chứa nhiều nguyên tắc oái ăm vô cùng. Nào là các cột phải là kiểu chuỗi, không cho viết hàm xâu chuỗi biến đổi dữ liệu. Sau một thời gian nghiên cứu, team front-end cũng bó tay. Ông leader bữa mới chém gió là thư viện này tiện ích này nọ mà giờ bỏ đi không xài nữa thì có nước độn thổ.

Việc gì đến cũng phải đến, thế là team back-end nhận được chỉ thị: "Tụi bay làm sao format dữ liệu đi. Đưa cho tụi front-end data chuẩn thôi. Tụi nó chỉ cần móc ra show lên thôi". ????

Nào là, "Bây giờ không chơi firstName, lastName nữa mà chơi 1 phát fullName thôi, cái dateOfBirth đừng để kiểu đó nữa, bay format thành string theo dd/MM/yyyy đi rồi trả ra" - leader phán. Ức chế nhất là: "Bây giờ cái Major trong Student, tụi bay làm sao hiển thị ra tên của Major trong Student luôn đi. Chứ cái bộ thư viện nó không cho tụi nó .major.name đâu".

![]({{site.baseurl}}/img/15.jpg)

Bây giờ có hai phương án để xử lý mấy cái yêu cầu này: 
- Một là mình sửa lại trong DB, gom hai cột firstName và lastName thành một, chỉnh kiểu của dat thành string và cuối cùng thêm một cột MajorName nữa. Tuy nhiên, phương án này bị loại đầu tiên, do phải đụng đến cấu trúc database để thống nhất từ trước. Chưa kể cái MajorName rất khó xử lý. Ví dụ trường hợp, người dùng đổi tên của Major, chúng ta lại phải update lại Major của toàn bộ sinh viên đang theo học chuyên ngành đó sao?
- Phương án hai là chúng ta sẽ format lại dữ liệu trong back-end. Chỉnh sửa API cho phù hợp và trả ra đúng kết quả mà tụi front-end cần. Và cuối cùng mình đã chọn áp dụng phương pháp này

Đầu tiên chúng ta tạo một Class StudentResource để chứa những thông tin và kiểu dữ liệu chuẩn

{% highlight csharp %}
    public class StudentResource
    {
        public int Id { get; set; }
        public string Code { get; set; }
        public string FullName { get; set; }
        public string Address { get; set; }
        public string Email { get; set; }
        public string Phone { get; set; }
        public string Year { get; set; }
        public string DateOfBirth { get; set; }
        public string MajorName { get; set; }
    }
{% endhighlight %}

Đồng thời chỉnh sửa lại API

{% highlight csharp %}
    [HttpGet]
    [Route("getAllStudent")]
    public IEnumerable<StudentResource> GetAllStudent()
    {
         var list = _context.Students
                    .Include(s => s.Major).AsEnumerable().Select(student => new StudentResource
                        {
                            Id = student.Id,
                            Code = student.Code,
                            FullName = student.LastName + " " + student.FirstName,
                            Address = student.Address,
                            Email = student.Email,
                            Phone = student.Phone,
                            Year = student.Year,
                            DateOfBirth = student.DateOfBirth.ToString("dd/MM/yyyy"),
                            MajorName = student.Major.Name
                        })
                    .ToList();

        return list;
    }
{% endhighlight %}

{% highlight json %}
[
   {
      "id":1,
      "code":"1331209041",
      "fullName":"Đàm Đức Duy",
      "address":"C57/18, khu 5, Chánh Nghĩa, Bình Dương",
      "email":"duy.dam.k3set@eiu.edu.vn",
      "phone":"09293841597",
      "year":"5",
      "dateOfBirth":"18/10/1995",
      "majorName":"Công nghệ thông tin"
   },
   {
      "id":2,
      "code":"1331209042",
      "fullName":"Nguyễn Văn Lang",
      "address":"Chợ Thủ Dầu Một, Bình Dương",
      "email":"hau.tran.k3set@eiu.edu.vn",
      "phone":"09293841597",
      "year":"5",
      "dateOfBirth":"18/07/1995",
      "majorName":"Công nghệ thông tin"
   }
]
{% endhighlight %}

OK, JSON sau khi call API cũng trả về rất là đẹp và ổn áp. Leader và team front-end vui mừng khen ngợi. Tuy nhiên, tự bản thân mình cảm giác không ổn. Với Student còn ít thông tin, tuy nhiên nếu một trường hợp một bảng khác với số lượng thông tin lớn thì code sẽ khá là dài. Có cách nào khá để giảm thiểu vấn đề này hoặc tránh bị duplicate code không, do có thể trong tương lai sẽ nhiều API cần trả về danh sách sinh viên nữa.

Đó là lý do chính khiến mình áp dụng thự viện AutoMapper. Lợi ích đầu nó mang lại chính là tự động map data giữa Student và StudentResource nếu có trường trùng tên và kiểu dữ liệu. Điều này có nghĩa, chúng ta không cần khai báo Id = student.Id hoặc Code = student.Code do Id và Code ở hai class trùng tên và trùng kiểu dữ liệu (với Id thì cả hai đều là int và code là string), AutoMapper sẽ tự động map data giữa những trường này. Việc này sẽ giúp chúng ta giảm thiểu độ dài cảu source code và dễ dàng trong việc quản lý. Tuy nhiên với trường hợp dateOfBirth thì sao, một bên là kiểu string trong khi bên kia là kiểu DateTime. Đây là trường hợp đặc biệt, yêu cầu một chút mẹo custome lại.

Đầu tiên, muốn áp dụng AutoMapper thì chúng ta phải add thư viện này vào. Có nhiều loại khác nhau, tuy nhiên mình hay dùng [AutoMapper.Extensions.Microsoft.DependencyInjection][AutoMapper.Extensions.Microsoft.DependencyInjection-link]. Các bạn có thể truy cập vào link để xem cách cài đặt. Sau khi cài đặt xong, các bạn vào Startup.cs, vào ConfigureServices sau đó thêm đoạn code này vào

{% highlight csharp %}
    var config = new AutoMapper.MapperConfiguration(cfg =>
    {
        cfg.AddProfile<MappingProfile>();
    });
    var mapper = config.CreateMapper();
    services.AddSingleton(mapper);
{% endhighlight %}

Nếu MappingProfile còn đang đỏ hoặc lỗi, thì các bạn đừng lo lắng, bây giờ chúng ta sẽ tạo nó ngay đây. Tạo MappingProfile.cs và dán đoạn code này vào file đó

{% highlight csharp %}
   public class MappingProfile : Profile
    {
        public MappingProfile()
        {
		    CreateMap<Student, StudentResource>()
                .ForMember(sr => sr.FullName, opt => opt.MapFrom(s => s.LastName + " " + s.FirstName))
                .ForMember(sr => sr.DateOfBirth, opt => opt.MapFrom(s => s.DateOfBirth.ToString("dd/MM/yyyy")))
                .ForMember(sr => sr.MajorName, opt => opt.MapFrom(s => s.Major.Name));
		}
	}
{% endhighlight %}

Giải thích sơ đoạn code ở trên. Chúng ta có thể nhận ra rằng đây chính là lệnh map data giữa object Student sang object StudentResource. Như trên mình đã nói, những trường có trùng tên và kiểu dữ liệu thì sẽ được tự động map. Những trường cần sự custom thì chúng ta sẽ custom trong hàm .ForMember(). Tương ứng với mỗi trường cần custom của StudentResource, data sẽ được MapFrom trường hoặc dữ liệu từ bên Student sao cho phù hợp với nhu cầu đề ra.

Đây mới chỉ là bước setup, tiếp theo chúng ta sẽ phải áp dụng thự viện này vào API đang viết ở trên. Bước đầu tiên, chúng ta phải nhúng AutoMapper vào constructor của Controller đã

{% highlight csharp %}
    private ApplicationDbContext _context;
    private IMapper _mapper;

    public StudentController(ApplicationDbContext context, IMapper mapper)
    {
        _context = context;
        _mapper = mapper;
    }
{% endhighlight %}

Sau đó, chỉ cần gọi hàm _mapper.Map là dữ liệu sẽ được tự động map giữa Student và StudentResource. Ở đây, do chúng ta cần xuất ra một List danh sách sinh viên nên trong cần phải khai báo cho AutoMapper là chúng ta map dữ liệu trong list. Việc còn lại thì cứ để AutoMapper xử lý theo cấu hình mà chúng ta đã khai báo bên MappingProfile. Code ngắn lại thấy rõ, ngoài ra nó còn rất dễ cập nhật và bảo trì, chỉ cần vào MappingProfile là có thể thay đổi cách map dữ liệu rồi.
{% highlight csharp %}
    HttpGet]
    [Route("getAllStudent")]
    public IEnumerable<StudentResource> GetAllStudent()
    {
        var list = _context.Students
                         .Include(s => s.Major)
                         .ToList();
        return _mapper.Map<List<Student>, List<StudentResource>>(list);
    }
{% endhighlight %}

JSON trả ra vẫn rất đẹp. Thế này team front-end vui lòng rồi.

{% highlight json %}
[
   {
      "id":1,
      "code":"1331209041",
      "fullName":"Đàm Đức Duy",
      "address":"C57/18, khu 5, Chánh Nghĩa, Bình Dương",
      "email":"duy.dam.k3set@eiu.edu.vn",
      "phone":"09293841597",
      "year":"5",
      "dateOfBirth":"18/10/1995",
      "majorName":"Công nghệ thông tin"
   },
   {
      "id":2,
      "code":"1331209042",
      "fullName":"Nguyễn Văn Lang",
      "address":"Chợ Thủ Dầu Một, Bình Dương",
      "email":"hau.tran.k3set@eiu.edu.vn",
      "phone":"09293841597",
      "year":"5",
      "dateOfBirth":"18/07/1995",
      "majorName":"Công nghệ thông tin"
   }
]
{% endhighlight %}

Trong phần 1 này, mình mới chỉ đề cập đến lí do, cách cài đặt và sử dụng HttpGet thôi. Trong phần hai, mình sẽ đi sâu hơn, và hướng dẫn các bạn cách áp dụng AutoMapper vàp HttpPost và HttpPut.
Hẹn gặp lại.

[automapper-docs]: https://automapper.org/
[AutoMapper.Extensions.Microsoft.DependencyInjection-link]: https://www.nuget.org/packages/AutoMapper.Extensions.Microsoft.DependencyInjection/
