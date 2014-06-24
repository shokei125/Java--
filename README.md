Java--
======
import java.io.BufferedReader;
import java.io.BufferedWriter;
import java.io.FileNotFoundException;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.*;

//countdown
class jikan extends Thread{
	public void run(){

	}
}
class gameTimer extends Thread{
	private int time;
	private int count;

	gameTimer(int t,int c){
		time = t;
		count = c;
	}

	public void run(){
		while(true){
			try{
				while(true){
					System.out.println(count);
					count--;
					Thread.sleep(time);
					if(count < 0) jikan.interrupted();
				}
			}  catch(InterruptedException e){
				break;
			}catch(Exception e){
			}

		}
	}
	public int getCount() {
	    return count;
	}
}

//quizbox
class QuizBox{
	String answer;
	String keyWord1;
	String keyWord2;

	public String getAnswer() {
	    return answer;
	}
	public void setAnswer(String answer) {
	    this.answer = answer;
	}
	public String getKeyWord1() {
	    return keyWord1;
	}
	public void setKeyWord1(String keyWord1) {
	    this.keyWord1 = keyWord1;
	}
	public String getKeyWord2() {
	    return keyWord2;
	}
	public void setKeyWord2(String keyWord2) {
	    this.keyWord2 = keyWord2;
	}

}

public class quiz {
	public static void main(String[] args) throws InterruptedException {

		String[] chooseBox = new String[4];//選択肢を入れる
		int[] chooseCheck = new int[4];//ランダム用
		int[] com = new int[100];//答えを入れる
		int[] play = new int[100];//player用
		int quizCont = 0;//問題数用
		int score = 0;//点数用
		int level = 0;//レベル用
		int tmp = 0;


		System.out.println("国当てゲーム");
		List<QuizBox> chooselist = csvToChooselist();
		String[] seikai = new String[chooselist.size()+1];

		String userId = nameIn();
		seikai[0] = userId;
		nameOut(userId);
		String[] headInfo = HeadInfo();
		if(nameCheck(userId)){
			System.out.println("クイズゲーム始まるぞ！！");
		}else {
			System.out.println("初めまして"+userId+"\nクイズゲーム始まるぞ！！");
			UserToCsv(headInfo, userId);//userの初期化
		}

		List<HashMap<String, String>> scorelist = csvToScorelist(userId);//userの情報の読み込み

		//ランダムで先に出題順番を決める
		for(int i = 0;i < chooselist.size();i++){
			com[i] = (int) Math.floor(Math.random()*chooselist.size());
			for(int j = 0;j < i;j++){
				if(com[j] == com[i] && j != i){
					i--;
					break;
				}
			}
		}

		//難易度判断
		level = levelChoose();
		switch(level){
			case 1: level = 5;break;
			case 2: level = 10;break;
			case 3: level = 15;break;
			default: level = chooselist.size()-1;
		}

		//------------ゲーム本体--------------//
		do{
			//keywordの表示
			String tmpA = chooselist.get(com[quizCont]).getKeyWord1();
			String tmpB = chooselist.get(com[quizCont]).getKeyWord2();
			keyExp(tmpA, tmpB);

			//選択肢の収納
			for(int i = 0;i < 4;i++){
				//quizboxからランダムの選択肢出す
				chooseCheck[i] = (int) Math.floor(Math.random()*chooselist.size());
				for(int j = 0;j < i;j++){
					if(chooseCheck[j] == chooseCheck[i] && i != j){
						i--;
						break;
					}
				}
				chooseBox[i] = chooselist.get(chooseCheck[i]).getAnswer();
			}

			//答え合わせ用
			String comer =chooselist.get(com[quizCont]).getAnswer();
			int numTmp = 0;
			//答えの収納
			for(int i = 0;i < 4;i++){
				//答えすでに入ってる場合
				if(chooseBox[i].equals(comer))
					break;
				numTmp++;
			}
			//答えが入ってない場合
			if(numTmp == 4) chooseBox[(int) Math.floor(Math.random()*4)] = comer;

			//選択肢の表示
			for(int i = 0;i < 4;i++)
				System.out.println((i+1)+"."+chooseBox[i]);
			int tmpp = 0;
			do{
				
				tmpp = 0;
				play[quizCont] = scanCheck()-1;
				if(play[quizCont] < 0 || play[quizCont] > 3){
					System.out.println("1-4の整数を入力してください！");
					tmpp++;
				}
			}while(tmpp != 0 );
				//答え合わせ
			if(answerCheck(chooseBox[play[quizCont]],comer)){
				System.out.println("正解です！！");
				score++;
				seikai[quizCont+1] = comer;
			}else{
				System.out.print("もう少し頑張りましょう！！");
			}
			quizCont++;

			System.out.println(seikai[0]+"の点数は"+score+"点です。");
			if(quizCont == level) break;
			System.out.print("【続く:1～9/ 終わる:0】");
			tmp = scanCheck();
		}while(tmp != 0);

		//---------点数の計算-------------------//
		//hashmapから配列に格納
		String[] key = new String[scorelist.size()];//hashmapのキーを格納★★★
		String[] data = new String[scorelist.size()];//hashmapのデータを格納★★★
		for(int i = 0;i < scorelist.size();i++){
			Set<String> k = scorelist.get(i).keySet();
			key[i] = String.valueOf(k);
			Collection<String> d = scorelist.get(i).values();
			data[i] = String.valueOf(d);
			data[i] = subGet(data[i]);
			if(scoreAdd(key[i],seikai)) {
				int val = Integer.parseInt(data[i]);
				val++;
				data[i] = String.valueOf(val);
			}
		}
		scoreOut(score);

		//-----------点数を修正してcsvに書き込む-------------//
		String userscore = subToString(data);
		//System.out.println(userscore);
		int userline = userLine(userId);//userのline
		String[] csvfile = csvGet();//csvfileの読み込み
		csvOut(csvfile,userscore,userline);

		//--------recommend--------//
		recommend.main(args);

		//-----------アドバイス収集-------------//
		thankU(userId);
		//-------------ゲームループ用---------//
		quiz.main(args);
		
		
		
		
		
		//タイマーの起動
		/*gameTimer test = new gameTimer(500,7);
		Thread jikan = new Thread();
		test.start();*/
	}

	//csvからヘッタ情報を読み込む
	public static String[] HeadInfo(){
		String[] header = null;
		try{
			FileReader fr = new FileReader("name.csv");
			BufferedReader br = new BufferedReader(fr);

			String line;
			int cont = 0;
			while((line = br.readLine()) != null){
				if(cont == 0){
					header = line.split(",",-1);
					cont++;
				}else break;
			}
			br.close();
		} catch (IOException ex){
			ex.printStackTrace();
		}
		return header;
	}

	//csvからファイルの全体を読み込む
	public static String[] csvGet(){
		List<String> alist = new ArrayList<String>();
		try{
			FileReader fr = new FileReader("name.csv");
			BufferedReader br = new BufferedReader(fr);
			String line;
			while((line = br.readLine()) != null){
				alist.add(line);
			}
			br.close();
		} catch(FileNotFoundException e){
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
		String[] csvFile = (String[])alist.toArray(new String[0]);
		return csvFile;
	}

	//csvから問題を読み込む
	public static List<QuizBox> csvToChooselist(){
		List<QuizBox> Chooselist = new ArrayList<QuizBox>();
		QuizBox choose = null;

		try {
			FileReader fr = new FileReader("quizz.csv");
			BufferedReader br = new BufferedReader(fr);

			String line;
			while((line = br.readLine()) != null){
				choose = new QuizBox();
				String[] row = line.split(",",-1);

				for(int i = 0;i < row.length; i++){
					switch(i){
						case 0: choose.setAnswer(row[i]);break;
						case 1: choose.setKeyWord1(row[i]);break;
						case 2: choose.setKeyWord2(row[i]);break;
					}
				}
				Chooselist.add(choose);
			}
			br.close();
		} catch (IOException ex){
			ex.printStackTrace();
		}
		return Chooselist;
	}

	//csvファイルのユーザーの存在の判断
	public static boolean nameCheck(String userId){
		String []row = null;
		try{
			FileReader fr = new FileReader("name.csv");
			BufferedReader br = new BufferedReader(fr);

			String line;
			while((line = br.readLine()) != null){
				row = line.split(",",-1);
				if(row[0].equals(userId)) {
					br.close();
					return true;
				}
			}
			br.close();
		}catch(IOException e){
			e.printStackTrace();
		}
		return false;
	}
	//userIdによってcsvからユーザーの情報を読み込む
	public static List<HashMap<String,String>> csvToScorelist(String userId){
		List<HashMap<String, String>> ScoreList = new ArrayList<HashMap<String,String>>();
		String[] header = null;
		String[] row = null;
		try {
			FileReader fr = new FileReader("name.csv");
			BufferedReader br = new BufferedReader(fr);

			String line;
			int cont = 0;
			while((line = br.readLine()) != null){
				if(cont == 0){
					header = line.split(",",-1);
					cont++;
				}else{
					row = line.split(",",-1);
					if(row[0].equals(userId)){
						row = line.split(",",-1);
						break;
					}
					else continue;
				}
			}
			br.close();
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
		//ユーザー情報の取得
		for(int i = 0;i < row.length;i++){
			HashMap<String, String> userScore = new HashMap<String, String>();
			userScore.put(header[i],row[i]);
			ScoreList.add(userScore);
		}
		return ScoreList;
	}

	//csvの中のuserの行数を取得
	public static int userLine(String userId){
		int cont = 0;
		String[] header = null;
		try {
			FileReader fr = new FileReader("name.csv");
			BufferedReader br = new BufferedReader(fr);

			String line;
			while((line = br.readLine()) != null){
				header = line.split(",",-1);
				if(header[0].equals(userId)) break;
				cont++;
			}
			br.close();
		} catch (FileNotFoundException e) {
			e.printStackTrace();
		} catch (IOException e) {
			e.printStackTrace();
		}
		return cont;
	}

	//新規ユーザーの初期化;
		public static void UserToCsv(String[] headInfo,String userId){
			int i = 0;
			try{
				 FileWriter fw = new FileWriter("name.csv", true);
		         PrintWriter pw = new PrintWriter(new BufferedWriter(fw));

				while(i < headInfo.length){
						if(i == 0) pw.write(userId+",");
						else if(i <headInfo.length-1) pw.write(0+",");
						else if(i == headInfo.length-1) pw.write(0+"\n");
						i++;
				}
				pw.close();
			}catch (IOException e){
				e.printStackTrace();
			}
		}

	//csvへのデータ更新の書き込み
	public static void csvOut(String[] csvfile,String userscore,int userline){
		int i = 0;
		try{
			FileWriter fw = new FileWriter("name.csv",false);
			PrintWriter pw = new PrintWriter(new BufferedWriter(fw));

			while(i < csvfile.length){
				if(i == userline) pw.write(userscore);
				else pw.write(csvfile[i]);
				pw.write("\n");
				i++;
			}
			pw.close();
		}catch (IOException e){
			e.printStackTrace();
		}
	}

	public static String nameIn(){
		String userId = null;
		int tmp;
		do{
			System.out.print("名前を入力してください：");
			userId = new Scanner(System.in).next();
			System.out.print("あなたの名前は"+userId+"です。\nこの名前でよろしですか?\nYes='1'  No=='9'");
			tmp = new Scanner(System.in).nextInt();
		}while(tmp != 1);
		return userId;
	}
	//入力チェック
	public static int scanCheck(){
		int tmp = 0;
		int numCheck = 0;
		do{
			try{
				tmp = 0;

					numCheck = new Scanner(System.in).nextInt();

			}catch (InputMismatchException e){
				System.out.println("1～４までの半角数字を入力してください！！");
				tmp++;
			}

		}while(tmp !=0);

		return numCheck;
	}

	//難易度の選択--問題数を選ぶこと
	public static int levelChoose(){
		int tmp = 0;
		int numCheck = 0;
		do{
			try{
				tmp = 0;
				System.out.println("難易度を選んでください。");
				System.out.println("「五問」--1/「十問」--2/「十五問」--3/「全問」--7");
				numCheck = new Scanner(System.in).nextInt();
			}catch (InputMismatchException e){
				System.out.println("整数を入力してください!");
				tmp++;
			}

		}while(tmp !=0);

		return numCheck;
	}

	//点数の計算
	public static boolean scoreAdd(String strA,String[] strB){
		for(int i = 0;i < strB.length;i++){
			if(strB[i] == null) continue;
			if(strA.equals("["+strB[i]+"]")) {
				return true;
			}
		}

		return false;
	}

	//keywordの表示
	public static void keyExp(String a,String b){
		System.out.println("------------------------------------");
		System.out.println("キーワード:\n"+"【"+a+"】\n"+"【"+b+"】");
		System.out.println("------------------------------------");
	}

	//答えのチェック
	public static boolean answerCheck(String c,String p){
		if(p.equals(c)) return true;
		return false;
	}

	//点数発表;
	public static void scoreOut(int s){

		System.out.println("あなたので点数"+s+"取りました");
	}

	//hashmapから"[]"を外す
	public static String subGet(String str){
		int size = str.length();
		str = str.substring(1,size-1);
		return str;
	}

	//String[]からStringへの変更
	public static String subToString(String[] str){
		StringBuilder bld = new StringBuilder();
		int i = 0;
		while(i < str.length){
			if(i == str.length-1){
				bld.append(str[i]);
				//bld.append("\n");
			}else{
				bld.append(str[i]);
				bld.append(",");
			}
			i++;
		}
		String userScore = bld.toString();
		return userScore;
	}

	//playerのnameをファイルに一時的に格納
	public static void nameOut(String name){
		try{
			FileWriter fw = new FileWriter("nameTmp.txt",false);
			PrintWriter pw = new PrintWriter(new BufferedWriter(fw));
			pw.write(name);
			pw.close();
		}catch (IOException e){
			e.printStackTrace();
		}
	}

	public static void thankU(String userId){
		System.out.println("よければ,\n"
				+ "ここに感想を書いてください:");
		String adv = new Scanner(System.in).next();
		try{
			FileWriter fw = new FileWriter("advirse.txt.",true);
			PrintWriter pw = new PrintWriter(new BufferedWriter(fw));
			pw.write(adv+"\n\r");
			pw.close();
		}catch (IOException e){
			e.printStackTrace();
		}
		System.out.println(userId+"さん、ご来場ありがとうございました！！！");
	}
}
