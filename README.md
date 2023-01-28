## Reference about text classification
- https://viblo.asia/p/feature-engineering-phan-4-phuong-phap-xu-ly-truyen-thong-voi-du-lieu-dang-van-ban-text-data-1Je5EvWYKnL
- https://viblo.asia/p/su-dung-rasa-custom-actions-xu-ly-cuoc-hoi-thoai-cho-chatbot-bJzKmOywl9N
### Rasa Reference 
- Custom rasa intent classification : https://github.com/MantisAI/rasa_custom_intent_classification
- Custom-form: 
  - https://github.com/AarohiSingla/ChatBot-Using-Rasa-2.0
  - https://learning.rasa.com/archive/rasa-forms-2/proper-name/
- Custom validate form : https://www.youtube.com/watch?v=md97kXT27Y0
- Action server api : https://rasa.com/docs/rasa/pages/action-server-api/
- Rasa server api : https://rasa.com/docs/rasa/pages/http-api
- Rasa doc:
  - Rasa 2x: https://rasa.com/docs/rasa/2.x
  - Rasa 3x: https://rasa.com/docs/rasa/
- Rasa webchatbot  
  - Rasa add carousels : 
    - https://www.youtube.com/watch?v=KVgIWuO-asE
    - https://gist.github.com/Horizon733/59f7e50cd2ea21c81e133996fdaa1ef2
- Rasa nlu-example
  - Rasa 2x: https://github.com/RasaHQ/rasa-nlu-examples/tree/0.2.8
  - Rasa 3x : https://github.com/RasaHQ/rasa-nlu-examples
  - Bayes rasa 2x: https://github.com/RasaHQ/rasa-nlu-examples/blob/0.2.8/rasa_nlu_examples/classifiers/sparse_naive_bayes_intent_classifier.py
## Cài đặt Rasa chatbot
### Phiên bản sử dụng
- Rasa 2.8.2
- Python 3.8
- virtualenv 20.13.2
### Cài đặt vituralenv ( sử dụng môi trường ảo )
- pip install virtualenv
## Coppy vào thư mục project các thư mục:
- lib_vi
- sparse_naive_bayes_intent_classifier
### Cài đặt môi trường ảo sử dụng python 3.8
- Tải phiên bản python 3.8 (Windows x86-64 executable installer) cài đặt chế độ custom ở một thư mục E:\folder\folder\folder
- tải thư viện python môi trường ảo : https://drive.google.com/drive/folders/1VfnApqPQqT6qyjF9p1vKADvKdUHShmYo?usp=share_link
- tạo một môi trường ảo: `python -m virtualenv -p  E:\folder\folder\folder\python38\python.exe py38`
- Kích hoạt môi trường ảo: thực hiện lệnh  `py38\Scripts\activate`
- Dừng môi trường ảo : thực hiện lệnh `py38\Scripts\deactivate`
## Các lớp trích xuất đặc trưng trong rasa
## Phobert
### Pipe-line recommend for phobert:
```
pipline: #phobert
   - name: VietnameseTokenizer
   - name: LanguageModelFeaturizer
     model_weights: "vinai/phobert-base"
     model_name: "phobert"
   - name: DIETClassifier
     epochs: 200
     constrain_similarities: true
```
## Fast-text: ( sử dụng các file trong thư mục fast_text )
- Tải thư mục từ điển tiếng việt (file bin) từ : https://fasttext.cc/docs/en/crawl-vectors.html
- Giải nén file vừa mới tải `cc.vi.300.bin.gz` được file có định dạng `cc.vi.300.bin`
- Trong file config rasa :
  - cache_dir là thư mục chứa file `cc.vi.300.bin`
## Pipe-line recommend for fast-text:
```
pipeline: 
   - name: VietnameseTokenizer
   - name: CountVectorsFeaturizer
     analyzer: char_wb
     min_ngram: 1
     max_ngram: 4
   - name: FastTextFeaturizer
     cache_dir: 'fasttext\vi'
     file: 'cc.vi.300.bin'
   - name: DIETClassifier
     epochs: 100
     constrain_similarities: true

```
## Byte-pair-encoding 
## Pipe-line recommend for bpemb:
```
pipeline: 
   - name: tùy chỉnh
   - name: BytePairFeaturizer
     lang: vi
     vs: 1000
     dim: 300
   - name: DIETClassifier
     epochs: 100
     constrain_similarities: true
```
## Mô hình phân lớp
## Pipe-line recommend for SVM:
```
pipeline:
   - name: VietnameseTokenizer
   - name: LanguageModelFeaturizer
     model_name: phobert
     model_weights: vinai/phobert-base
   - name: "SklearnIntentClassifier"
    # Specifies the list of regularization values to
    # cross-validate over for C-SVM.
    # This is used with the ``kernel`` hyperparameter in GridSearchCV.
     C: [1, 2, 5, 10, 20, 100]
    # Specifies the kernel to use with C-SVM.
    # This is used with the ``C`` hyperparameter in GridSearchCV.
     kernels: ["linear","poly","sigmoid","rbf"]
    # Gamma parameter of the C-SVM.
     "gamma": [0.1]
    # We try to find a good number of cross folds to use during
    # intent training, this specifies the max number of folds.
     "max_cross_validation_folds": 5
    # Scoring function used for evaluating the hyper parameters.
    # This can be a name or a function.
     "scoring_function": "f1_weighted"
   - name: DIETClassifier
     epochs: 100
     intent_classification: False # tắt chức năng phân lớp của diet chỉ dùng nhận dạng thực thể
     constrain_similarities: true
```
## Pipe-line recommend for BAYES
- file sparse_naive_bayes_intent_classifier để trong thư mục project rasa
```
pipeline:
   - name: VietnameseTokenizer
   - name: CountVectorsFeaturizer
   - name: CountVectorsFeaturizer
     analyzer: char_wb
     min_ngram: 1
     max_ngram: 4
   - name: sparse_naive_bayes_intent_classifier.SparseNaiveBayesIntentClassifier
     alpha: 0.1
   - name: DIETClassifier
     epochs: 100
     intent_classification: False
     constrain_similarities: true
```
## Đánh giá các mô hình
- dùng lệnh: rasa test nlu --nlu data/nlu.yml --cross-validation --config config/file_cấu_hình.yml
- phương pháp sử dụng phương thức kfold
- k mặc định = 5 
