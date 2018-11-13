import java.io.FileInputStream;
import java.io.FileNotFoundException;
import java.util.Scanner;
import java.util.Map.Entry;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.LinkedBlockingQueue;

public class ReadAndAnalysisFile2 {

	ConcurrentHashMap<String, String[]> map = new ConcurrentHashMap<>();
	LinkedBlockingQueue<String> bystoplines = new LinkedBlockingQueue<>();
	LinkedBlockingQueue<String> bagnolines = new LinkedBlockingQueue<>();

	public static void main(String[] args) {
		ReadAndAnalysisFile2 readAndAnalysisFile2 = new ReadAndAnalysisFile2();
		readAndAnalysisFile2.new ReadThread().start();
		readAndAnalysisFile2.new BystopAnalysis().start();
		readAndAnalysisFile2.new BagnoAnalysis().start();
	}

	class ReadThread extends Thread {
		@Override
		public void run() {
			String line = null;
			try (Scanner scanner = new Scanner(new FileInputStream("FDSdata2018new.txt"))) {
				while (scanner.hasNextLine()) {
					line = scanner.nextLine();
					if (line.contains("apcd")) {
						bystoplines.put(line);
						// System.out.println("acpdReading " + line);
					}
					if (line.contains("btno")) {
						bagnolines.put(line);
						// System.out.println("btnoReading " + line);
					}

				}
				bystoplines.put("stop");
				bagnolines.put("stop");
			} catch (FileNotFoundException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}

	// 分析航班经停
	class BystopAnalysis extends Thread {
		@Override
		public void run() {
			String line = null;
			while (true) {
				try {
					line = bystoplines.take();
					if (line.equals("stop"))
						break;
					
					// System.out.println("acpdAnalyzing " + line);
					
					// 以下为读取航班号
//					int index = line.indexOf("ffid");
//					index = index + 5;
//					String tmp1 = line.substring(index, index + 2);
//					String tmp2 = line.substring(index + 3);
//					tmp2 = tmp2.split("-")[0];
//					String flno = tmp1.concat(tmp2);
//
//					//以下为读取经过站点
//					//value字符串数字分别表示 经停站 行李转盘号和更新状态
//					String[] value = new String[3];
//					value[0] = "";
//					String tmpapcd = line;
//					while (tmpapcd.indexOf("apcd") != -1) {
//
//						index = tmpapcd.indexOf("apcd");
//						index = index + 5;
//						tmpapcd = tmpapcd.substring(index);
//						value[0] = value[0] + tmpapcd.split(",")[0] + "/";
//
//					}
//					//更新状态为未更新  行李转盘号为空
//					value[2] = "未更新";
//					value[1] = "    ";
//					
//					// 如果没存过
//					if (!map.containsKey(flno)) {
//						map.put(flno, value);
//						System.out
//								.println("1航班号：" + flno + "经停站：" + value[0] + "行李转盘号：" + value[1] + "更新状态：" + value[2]);
//						
//					}
//					
//					// 如果有新数据
//					if (!value[0].equals(map.get(flno)[0])) {
//						// 在 Java 中，如果要比较 a 字符串是否等于 b 字符串，需要这么写： if(a.equals(b)) 不能写==
//
//						System.out.println(value[0] + "对比" + map.get(flno)[0]);
//						System.out.println("1changebefore" + "航班号：" + flno + "经停站：" + map.get(flno)[0] + "行李转盘号："
//								+ map.get(flno)[1] + "更新状态：" + map.get(flno)[2]);
//						String tmp[] = new String[3];
//						tmp = map.get(flno);
//
//						if (value[2].equals("未更新"))
//							value[2] = "更新";
//						else {
//							value[2] = tmp[2] + "更新";
//						}
//
//						map.replace(flno, tmp, value);
//						System.out.println("1changeafter" + "航班号：" + flno + "经停站：" + map.get(flno)[0] + "行李转盘号："
//								+ map.get(flno)[1] + "更新状态：" + map.get(flno)[2]);
//
//					}

				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		}

	}

	// 分析航班经停
	class BagnoAnalysis extends Thread {
		@Override
		public void run() {
			String line = null;
			while (true) {
				try {
					line = bagnolines.take();
					if (line.equals("stop"))
						break;
					System.out.println(line);
					// 以下为读取航班号
					int index = line.indexOf("ffid");
					index = index + 5;
					String tmp1 = line.substring(index, index + 2);
					String tmp2 = line.substring(index + 3);
					tmp2 = tmp2.split("-")[0];
					String flno = tmp1.concat(tmp2);
					// System.out.println("航班号：" + flno);
					
					// 以下为读取行李转盘号
					String[] value = new String[3];
					value[1] = "";				
					String tmpapcd = line;					
					while (tmpapcd .indexOf("btno") != -1) {											 
						index = tmpapcd.indexOf("code");
						tmpapcd = tmpapcd.substring(index + 5);
						tmp1 = tmp1.split(",")[0];
						value[1] = value[1] + tmpapcd.split(",")[0] + "/";
					// System.out.println("行李转盘号：" + tmp1);
					}
					
					//更新状态为未更新  经停站为空
					value[2] = "未更新";
					value[0] = "    ";
						
					// 以下将数据加入hashmap
					if (!map.containsKey(flno)) {
						map.put(flno, value);
						System.out
						.println("2航班号：" + flno + "经停站：" + value[0] + "行李转盘号：" + value[1] + "更新状态：" + value[2]);

					}  
					if (!value[1].equals(map.get(flno)[1])){
						System.out.println("2changebefore" + "航班号：" + flno + "经停站：" + map.get(flno)[0] + "行李转盘号："
								+ map.get(flno)[1] + "更新状态：" + map.get(flno)[2]);
						String tmp[] = new String[2];
						tmp = map.get(flno);
						value[0] = tmp[0];
						
						if (value[2].equals("未更新"))
							value[2] = "更新";
						else {
							value[2] = tmp[2] + "更新";
						}
						
						map.replace(flno, tmp, value);
						System.out.println("2changeafter" + "航班号：" + flno + "经停站：" + map.get(flno)[0] + "行李转盘号："
								+ map.get(flno)[1] + "更新状态：" + map.get(flno)[2]);

					}
				} catch (Exception e) {
					e.printStackTrace();
				}
			}
		}

	}
	// end

}
