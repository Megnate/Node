```java
package javaEE.suanfa;

import java.util.*;

/**
 * 主要是介绍list集合的十种排序方法
 */
public class List_sort {
    /**
     * 单属性变量 list，根据自身属性进行排序
     */
    private void singleVariableSort1(){
        //List<Integer> list = new ArrayList<Integer>(){10,1,6,4,8,7,9,3,2,5};
        // 使用这种办法将数组转换成 List集合，有一个坏处：没法使用 add(),remove(),clear()等方法，所以无法动态改变list的长度
        // 如果仅仅只是用于遍历，就可以直接使用这个，不然就还是一个一个添加吧
        List<Integer> list = Arrays.asList(10,1,6,4,8,7,9,3,2,5);
        list.forEach(System.out::println);
//        list.forEach(n -> {
//            System.out.println(n)
//        });
        //这个代码等价于上面的System.out::println，将遍历出来的对象都去调用 System.out 的 println方法
        //升序排列
        Collections.sort(list);
        list.forEach(System.out::println);
        //倒序排列
        Collections.reverse(list);
    }

    /**
     * 根据自定义的顺序进行排序，也就是定义 Comparator，重新定义排序的规则
     */
    private void singleVariableSort2(){
        List<String> list = Arrays.asList("北京","上海","北京","广州","广州","上海","北京","上海");

        List<String> sortRule = Arrays.asList("北京","上海","广州");

        // 下面这个代码也可以这样写：Collections.sort(list, new Comparator<String>(){});
        list.sort((o1, o2) -> {
            int io1 = sortRule.indexOf(o1);
            int io2 = sortRule.indexOf(o2);
            return io1 - io2;
        });
        list.forEach(System.out::println);

        //倒序排序，这里是没有采用lambda的形式，是完整的形式，但是逻辑和上面的是一样的
        list.sort(new Comparator<String>() {
            @Override
            public int compare(String o1, String o2) {
                int io1 = sortRule.indexOf(o1);
                int io2 = sortRule.indexOf(o2);
                return io2 - io1;
            }
        });
        list.forEach(System.out::println);
    }

    /**
     * 按照对象单属性进行排序
     */
    private void entitySort1(){
        List<Student> students = new ArrayList<>();
        students.add(new Student("张三",90,180,"电气学院","北京"));
        students.add(new Student("李四",80,165,"计算机学院","上海"));
        students.add(new Student("王五",91,170,"财经学院","上海"));
        students.add(new Student("赵明",80,182,"计算机学院","北京"));
        students.add(new Student("钱海",75,181,"计算机学院","广州"));
        students.add(new Student("孙理",82,172,"财经学院","上海"));
        students.add(new Student("周伟",90,168,"电气学院","广州"));
        students.add(new Student("郑亮",80,178,"财经学院","广州"));

        //按照分数进行升序
        students.sort(Comparator.comparing(Student::getScore));
        students.forEach(s -> {
            System.out.println(s.getName()+" "+ s.getScore()+" "+s.getHeight()+" "+ s.getCollege()+""+s.getAddress());
        });

        //按照分数进行降序
        students.sort(Comparator.comparing(Student::getScore).reversed());
    }

    /**
     * 按照多属性进行排序
     */
    private void entitySort2(){
        List<Student> students = new ArrayList<>();
        students.add(new Student("张三",90,180,"电气学院","北京"));
        students.add(new Student("李四",80,165,"计算机学院","上海"));
        students.add(new Student("王五",91,170,"财经学院","上海"));
        students.add(new Student("赵明",80,182,"计算机学院","北京"));
        students.add(new Student("钱海",75,181,"计算机学院","广州"));
        students.add(new Student("孙理",82,172,"财经学院","上海"));
        students.add(new Student("周伟",90,168,"电气学院","广州"));
        students.add(new Student("郑亮",80,178,"财经学院","广州"));

        //在按照分数进行降序排序，当分数相同的时候，按照身高进行升序排序
//        students.sort(Comparator.comparing(Student::getScore).reversed().thenComparing(Student::getHeight));
//        students.forEach(s -> {
//            System.out.println(s.getName()+" "+ s.getScore()+" "+s.getHeight()+" "+ s.getCollege()+""+s.getAddress());
//        });

        //第二种写法，采用比较器的方式，这里可以使用lambda的方式进行缩写
        students.sort((o1, o2) -> {
            if (o1.getScore() == o2.getScore()) {
                return o2.getHeight() - o1.getHeight();
            } else {
                return o2.getScore() - o1.getScore();
            }
        });
        students.forEach(s -> {
            System.out.println(s.getName()+" "+ s.getScore()+" "+s.getHeight()+" "+ s.getCollege()+""+s.getAddress());
        });
    }

    /**
     * 根据自定的单属性顺序进行排序
     */
    private void entitySort3(){
        List<Student> students = new ArrayList<>();
        students.add(new Student("张三",90,180,"电气学院","北京"));
        students.add(new Student("李四",80,165,"计算机学院","上海"));
        students.add(new Student("王五",91,170,"财经学院","上海"));
        students.add(new Student("赵明",80,182,"计算机学院","北京"));
        students.add(new Student("钱海",75,181,"计算机学院","广州"));
        students.add(new Student("孙理",82,172,"财经学院","上海"));
        students.add(new Student("周伟",90,168,"电气学院","广州"));
        students.add(new Student("郑亮",80,178,"财经学院","广州"));

        //自定义按照地区的顺序进行排序：北京，上海，广州
        List<String> Address_list = Arrays.asList("北京","上海","广州");
        students.sort((o1, o2) -> {
            int io1 = Address_list.indexOf(o1.getAddress());
            int io2 = Address_list.indexOf(o2.getAddress());
            return io1 - io2;
        });
        students.forEach(s -> {
            System.out.println(s.getName()+" "+ s.getScore()+" "+s.getHeight()+" "+ s.getCollege()+" "+s.getAddress());
        });
    }

    /**
     * 根据自定义多属性进行排序
     */
    private void entitySort4(){
        List<Student> students = new ArrayList<>();
        students.add(new Student("张三",90,180,"电气学院","北京"));
        students.add(new Student("李四",80,165,"计算机学院","上海"));
        students.add(new Student("王五",91,170,"财经学院","上海"));
        students.add(new Student("赵明",80,182,"计算机学院","北京"));
        students.add(new Student("钱海",75,181,"计算机学院","广州"));
        students.add(new Student("孙理",82,172,"财经学院","上海"));
        students.add(new Student("周伟",90,168,"电气学院","广州"));
        students.add(new Student("郑亮",80,178,"财经学院","广州"));

        //自定义按照地区的顺序进行排序：北京，上海，广州
        List<String> Address_list = Arrays.asList("北京","上海","广州");
        //自定义按照学院进行排序：计算机学院，电气学院，财经学院
        List<String> College_list = Arrays.asList("计算机学院","电气学院","财经学院");

        students.sort((o1, o2) -> {
            int io1;
            int io2;
            if (o1.getCollege().equals(o2.getCollege())){
                io1 = Address_list.indexOf(o1.getAddress());
                io2 = Address_list.indexOf(o2.getAddress());
            }else {
                io1 = College_list.indexOf(o1.getCollege());
                io2 = College_list.indexOf(o2.getCollege());
            }
            return io1-io2;
        });
        students.forEach(s -> {
            System.out.println(s.getName()+" "+ s.getScore()+" "+s.getHeight()+" "+ s.getCollege()+" "+s.getAddress());
        });
    }
    private static class Student{
        private String name;
        private int score;
        private int height;
        private String college;
        private String address;

        private Student(){};

        public Student(String name, int score, int height, String college, String address) {
            this.name = name;
            this.score = score;
            this.height = height;
            this.college = college;
            this.address = address;
        }

        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            Student student = (Student) o;
            return score == student.score &&
                    height == student.height &&
                    Objects.equals(name, student.name) &&
                    Objects.equals(college, student.college) &&
                    Objects.equals(address, student.address);
        }

        public String getName() {
            return name;
        }

        public void setName(String name) {
            this.name = name;
        }

        public int getScore() {
            return score;
        }

        public void setScore(int score) {
            this.score = score;
        }

        public int getHeight() {
            return height;
        }

        public void setHeight(int height) {
            this.height = height;
        }

        public String getCollege() {
            return college;
        }

        public void setCollege(String college) {
            this.college = college;
        }

        public String getAddress() {
            return address;
        }

        public void setAddress(String address) {
            this.address = address;
        }
    }
    public static void main(String[] args) {
        List_sort ls = new List_sort();
        ls.entitySort4();
    }
}

```

