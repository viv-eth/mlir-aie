##===- Makefile -----------------------------------------------------------===##
# 
# This file licensed under the Apache License v2.0 with LLVM Exceptions.
# See https://llvm.org/LICENSE.txt for license information.
# SPDX-License-Identifier: Apache-2.0 WITH LLVM-exception
#
# Copyright (C) 2024, Advanced Micro Devices, Inc.
# 
##===----------------------------------------------------------------------===##

srcdir := $(shell dirname $(realpath $(firstword $(MAKEFILE_LIST))))

include ${srcdir}/../../../makefile-common

tensor_height = 8
tensor_width = 8
tile_height = 2
tile_width = 2
data_str=${tensor_height}_${tensor_width}_${tile_height}_${tile_width}
aie_py_src=tile_group.py

.PHONY: all template clean

all: build/final_${data_str}.xclbin

build/aie_${data_str}.mlir: ${srcdir}/${aie_py_src}
	mkdir -p ${@D}
	python3 $< --tensor-height ${tensor_height} --tensor-width ${tensor_width} --tile-height ${tile_height} --tile-width ${tile_width} > $@

build/final_${data_str}.xclbin: build/aie_${data_str}.mlir
	mkdir -p ${@D}
	cd ${@D} && aiecc.py --aie-generate-xclbin --aie-generate-npu-insts --no-compile-host \
		--no-xchesscc --no-xbridge \
		--xclbin-name=${@F} --npu-insts-name=insts_${data_str}.bin $(<:%=../%)

run_py: build/final_${data_str}.xclbin build/insts_${data_str}.bin
	${powershell} python3 ${srcdir}/test.py -x build/final_${data_str}.xclbin -i build/insts_${data_str}.bin -k MLIR_AIE --tensor-height ${tensor_height} --tensor-width ${tensor_width} --tile-height ${tile_height} --tile-width ${tile_width}

generate_access_map: ${srcdir}/${aie_py_src}
	mkdir -p ${@D}
	python3 $< --tensor-height ${tensor_height} --tensor-width ${tensor_width} --tile-height ${tile_height} --tile-width ${tile_width} --generate-access-map ${M} ${K}

clean:
	rm -rf build
