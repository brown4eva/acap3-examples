DEPS = liblarod
CFLAGS += $(shell PKG_CONFIG_PATH=$(PKG_CONFIG_PATH) pkg-config \
            --cflags $(DEPS))
LDLIBS += $(shell PKG_CONFIG_PATH=$(PKG_CONFIG_PATH) pkg-config \
            --libs $(DEPS))
CFLAGS += -Wall -Wextra -Werror

.PHONY: all
all: larod_simple_app model/mobilenet_v1_1.0_224_quant.larod

model/mobilenet_v1_1.0_224_quant.larod:
	mkdir -p model
	curl -o model/mobilenet_v1_1.0_224_quant.tgz http://download.tensorflow.org/models/mobilenet_v1_2018_08_02/mobilenet_v1_1.0_224_quant.tgz
	tar -xvf model/mobilenet_v1_1.0_224_quant.tgz -C model
	larod-convert.py tflite model/mobilenet_v1_1.0_224_quant.tflite
	rm -f model/*.tgz model/*.tflite model/*.pb* model/*.ckpt* model/*.txt
	curl -L -o model/labels_mobilenet_quant_v1_224.txt https://github.com/google-coral/edgetpu/raw/master/test_data/imagenet_labels.txt

.PHONY: clean
clean:
	-mv package.conf.orig package.conf
	rm -f larod_simple_app *.o *.eap *_LICENSE.txt
	rm -rf model
