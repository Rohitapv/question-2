# question-2
Build a word count application, where the constraints are that you have 10 MB RAM and 1 GB text file. You should be able to efficiently parse the text file and output the words and counts in a sorted way. Write a program to read a large file, and emit the sorted words along with the count. 
import java.io.BufferedReader;
import java.io.FileReader;
import java.io.IOException;
import java.util.HashMap;
import java.util.Map;
import java.util.PriorityQueue;

class TrieNode {
    Map<Character, TrieNode> children = new HashMap<>();
    boolean isEndOfWord;
    int count;
}

public class WordCountApplication {

    private static TrieNode root = new TrieNode();

    private static void insertWord(String word) {
        TrieNode node = root;
        for (char ch : word.toCharArray()) {
            node.children.putIfAbsent(ch, new TrieNode());
            node = node.children.get(ch);
        }
        node.isEndOfWord = true;
        node.count++;
    }

    private static void buildTrieFromFile(String filePath) throws IOException {
        try (BufferedReader reader = new BufferedReader(new FileReader(filePath))) {
            String line;
            while ((line = reader.readLine()) != null) {
                String[] words = line.toLowerCase().split("\\s+");
                for (String word : words) {
                    word = word.replaceAll("[^a-zA-Z]", ""); // Remove non-alphabetic characters
                    if (!word.isEmpty()) {
                        insertWord(word);
                    }
                }
            }
        }
    }

    private static void dfs(TrieNode node, String prefix, PriorityQueue<Map.Entry<String, Integer>> results) {
        if (node.isEndOfWord) {
            results.offer(Map.entry(prefix, node.count));
        }

        for (Map.Entry<Character, TrieNode> child : node.children.entrySet()) {
            dfs(child.getValue(), prefix + child.getKey(), results);
        }
    }

    private static PriorityQueue<Map.Entry<String, Integer>> wordCountAndFuzzySearch(String query, int k) {
        TrieNode node = root;
        for (char ch : query.toCharArray()) {
            if (node.children.containsKey(ch)) {
                node = node.children.get(ch);
            } else {
                return new PriorityQueue<>(); // No results found
            }
        }

        PriorityQueue<Map.Entry<String, Integer>> results = new PriorityQueue<>((a, b) -> b.getValue() - a.getValue());
        dfs(node, query, results);
        return results;
    }

    public static void main(String[] args) {
        try {
            String filePath = "your_large_text_file.txt";
            buildTrieFromFile(filePath);

            while (true) {
                System.out.print("Enter a word or prefix to search (type 'exit' to quit): ");
                String query = System.console().readLine().toLowerCase();
                if (query.equals("exit")) {
                    break;
                }

                PriorityQueue<Map.Entry<String, Integer>> wordCountResults = wordCountAndFuzzySearch(query, 10);

                if (wordCountResults.isEmpty()) {
                    System.out.println("No results found.");
                } else {
                    System.out.println("Word\tCount");
                    System.out.println("--------------");
                    for (Map.Entry<String, Integer> result : wordCountResults) {
                        System.out.println(result.getKey() + "\t" + result.getValue());
                    }
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
